# Polkadart — Dart/Flutter SDK for Polkadot

## Overview

Polkadart is a comprehensive Dart/Flutter SDK for Polkadot and Substrate-based blockchains. It enables cross-platform dApp development targeting iOS, Android, Web, macOS, Windows, and Linux from a single codebase.

**Use Polkadart when building mobile or Flutter dApps on Polkadot.** For web/Node.js projects, use PAPI or Dedot instead.

Key features:
- Type-safe generated APIs via `polkadart_cli` — full autocomplete, no stringly-typed calls
- Full native crypto in Dart: sr25519, ed25519, ecdsa (no FFI bridge needed)
- ink! smart contract support via `ink_cli`
- Truly cross-platform: iOS, Android, Web, macOS, Windows, Linux
- Web3 Foundation funded — [github.com/justkawal/polkadart](https://github.com/justkawal/polkadart)
- Community: [#polkadart:matrix.org](https://matrix.to/#/#polkadart:matrix.org)

## Installation

Add dependencies to your `pubspec.yaml`:

```yaml
dependencies:
  polkadart: ^0.7.0
  polkadart_keyring: ^0.7.0

dev_dependencies:
  polkadart_cli: ^0.7.0
```

Then add the chain configuration (still in `pubspec.yaml`) that tells `polkadart_cli` which chains to generate types for:

```yaml
polkadart:
  output_dir: lib/generated
  chains:
    polkadot: wss://rpc.polkadot.io
    kusama: wss://kusama-rpc.polkadot.io
    westend: wss://westend-rpc.polkadot.io
```

Run `flutter pub get` (or `dart pub get`) to install packages.

## Type Generation

Generate strongly-typed chain bindings before writing any chain interaction code:

```bash
dart run polkadart_cli:generate -v
```

This connects to each configured chain endpoint, fetches its runtime metadata, and generates typed Dart classes in `lib/generated/`. **Commit the generated files** — they are your typed chain bindings.

```
lib/generated/
  polkadot/
    polkadot.dart        # Main entry point
    query/               # Storage query classes
    tx/                  # Extrinsic builders
    types/               # Runtime type definitions
  kusama/
    kusama.dart
    ...
```

Re-run generation after a runtime upgrade to pick up new types.

## Connect to a Chain

```dart
import 'package:polkadart/polkadart.dart';
import './generated/polkadot/polkadot.dart';

// Connect via WebSocket
final provider = Provider.fromUri(Uri.parse('wss://rpc.polkadot.io'));
final polkadot = Polkadot(provider);

// Connect to Kusama
import './generated/kusama/kusama.dart';
final kusamaProvider = Provider.fromUri(Uri.parse('wss://kusama-rpc.polkadot.io'));
final kusama = Kusama(kusamaProvider);

// Always disconnect when done
await provider.disconnect();
```

## Read Storage

```dart
import 'package:polkadart/polkadart.dart';
import 'package:polkadart_keyring/polkadart_keyring.dart';
import './generated/polkadot/polkadot.dart';

final provider = Provider.fromUri(Uri.parse('wss://rpc.polkadot.io'));
final polkadot = Polkadot(provider);

// Query account info (balance, nonce, etc.)
final accountInfo = await polkadot.query.system.account(account.pubkey);
print('Free balance: ${accountInfo.data.free}');
print('Reserved:     ${accountInfo.data.reserved}');
print('Nonce:        ${accountInfo.nonce}');

// Query total issuance
final totalIssuance = await polkadot.query.balances.totalIssuance();
print('Total issuance: $totalIssuance');

// Read a runtime constant
final existentialDeposit = polkadot.consts.balances.existentialDeposit;
print('Existential deposit: $existentialDeposit');
```

## Subscribe to Changes

```dart
// Subscribe to account balance changes — returns a StreamSubscription
final subscription = polkadot.query.system
    .accountStream(account.pubkey)
    .listen((accountInfo) {
  print('Balance updated: ${accountInfo.data.free}');
});

// Cancel when no longer needed
await subscription.cancel();
```

## Submit Transactions

Polkadart uses an explicit sign → encode → submit flow:

```dart
import 'package:polkadart/polkadart.dart';
import 'package:polkadart_keyring/polkadart_keyring.dart';
import './generated/polkadot/polkadot.dart';

final provider = Provider.fromUri(Uri.parse('wss://rpc.polkadot.io'));
final polkadot = Polkadot(provider);

// Load a keypair
final wallet = await KeyPair.sr25519.fromUri('//Alice');

// Fetch chain state needed for the signing payload
final runtimeVersion = await polkadot.rpc.state.getRuntimeVersion();
final nonce = await polkadot.rpc.system.accountNextIndex(wallet.address);
final blockHash = await polkadot.rpc.chain.getBlockHash();
final genesisHash = await polkadot.rpc.chain.getBlockHash(blockNumber: 0);

// Build the call
final multiAddress = MultiAddress.id(destAddress);
final transferCall = polkadot.tx.balances
    .transferKeepAlive(dest: multiAddress, value: BigInt.one)
    .encode();

// Build signing payload
final payload = SigningPayload(
  method: transferCall,
  specVersion: runtimeVersion.specVersion,
  transactionVersion: runtimeVersion.transactionVersion,
  genesisHash: genesisHash,
  blockHash: blockHash,
  blockNumber: 0,
  eraPeriod: 64,
  nonce: nonce,
  tip: BigInt.zero,
).encode(polkadot.registry);

// Sign
final signature = wallet.sign(payload);

// Encode the extrinsic
final extrinsic = ExtrinsicPayload(
  signer: wallet.bytes(),
  method: transferCall,
  signature: signature,
  eraPeriod: 64,
  blockHash: blockHash,
  blockNumber: 0,
  nonce: nonce,
  specVersion: runtimeVersion.specVersion,
  tip: BigInt.zero,
  transactionVersion: runtimeVersion.transactionVersion,
).encode(polkadot.registry, SignatureType.sr25519);

// Submit and watch
final author = AuthorApi(provider);
await author.submitAndWatchExtrinsic(extrinsic, (ExtrinsicStatus data) {
  print('Extrinsic status: $data');
});
```

## Keyring

`polkadart_keyring` provides full native Dart crypto — no FFI or platform channels required:

```dart
import 'package:polkadart_keyring/polkadart_keyring.dart';

// Generate from mnemonic (Sr25519)
final wallet = await KeyPair.sr25519.fromMnemonic(
  'bottom drive obey lake curtain smoke basket hold race lonely fit walk',
);

// Generate from URI (dev accounts)
final alice = await KeyPair.sr25519.fromUri('//Alice');
final bob   = await KeyPair.sr25519.fromUri('//Bob');
final child = await KeyPair.sr25519.fromUri('//Alice//stash');

// Ed25519 and ECDSA are also supported
final edWallet   = await KeyPair.ed25519.fromMnemonic(mnemonic);
final ecdsaWallet = await KeyPair.ecdsa.fromMnemonic(mnemonic);

// SS58 address encoding
print('Address: ${wallet.address}'); // Polkadot (prefix 0)
print('Kusama:  ${wallet.address}'); // Use SS58 library for other prefixes

// Sign arbitrary bytes
final signature = wallet.sign(Uint8List.fromList(message));

// Verify
final valid = wallet.verify(message, signature);
```

## Smart Contracts

Polkadart supports ink! smart contracts via `ink_cli` and `ink_abi`:

```dart
import 'package:polkadart/polkadart.dart';
import 'package:ink_abi/ink_abi.dart';

// Load contract ABI
final abi = ContractAbi.fromJson(abiJson);

// Instantiate contract client
final contract = Contract(
  provider: provider,
  abi: abi,
  address: contractAddress,
);

// Dry-run a query (no gas consumed)
final result = await contract.call(
  origin: wallet.address,
  method: 'balanceOf',
  args: [wallet.address],
);
print('Token balance: ${result.output}');

// Send a transaction
final call = contract.buildCall(method: 'transfer', args: [dest, amount]);
// Encode and submit via AuthorApi (same pattern as regular extrinsics)
```

## Packages

Polkadart is a monorepo. Here are the key packages:

| Package | Description |
|---------|-------------|
| `polkadart` | Core provider, RPC, storage queries, extrinsic submission |
| `polkadart_keyring` | Key generation, signing — sr25519, ed25519, ecdsa |
| `polkadart_cli` | Code generator — fetches metadata and produces typed Dart classes |
| `polkadart_scale_codec` | SCALE encoding/decoding |
| `substrate_metadata` | Runtime metadata parsing (V14/V15) |
| `ink_abi` | ink! ABI parsing and encoding |
| `ink_cli` | ink! contract interaction CLI tools |
| `sr25519` | Native Dart sr25519 implementation |
| `secp256k1_ecdsa` | Native Dart secp256k1 ECDSA |
| `ss58` | SS58 address encoding/decoding |
| `substrate_bip39` | BIP39 mnemonic support for Substrate |

## Best Practices

1. **Commit generated types** — The output of `dart run polkadart_cli:generate` belongs in source control; it is your typed chain interface
2. **Regenerate after runtime upgrades** — When a chain does a runtime upgrade, re-run the generator and commit the diff; stale types will cause encoding errors
3. **Use `BigInt` for balances** — Polkadot balances are `u128`; always use Dart's `BigInt`, never `int` (which is 64-bit and will overflow)
4. **Disconnect providers** — Always call `provider.disconnect()` in `dispose()` or when the widget/screen is removed
5. **Fetch nonce fresh per transaction** — Never cache or reuse a nonce; always fetch via `accountNextIndex` immediately before building the payload
6. **Use Sr25519 by default** — Sr25519 is the standard key type on Polkadot/Substrate; use Ed25519 or ECDSA only when the chain or use case requires it
7. **Test on Westend first** — Westend is the official Polkadot testnet with a free faucet; validate your transaction logic there before hitting mainnet
8. **Handle stream cleanup** — Store `StreamSubscription` references and call `cancel()` on disposal to prevent memory leaks in Flutter apps
