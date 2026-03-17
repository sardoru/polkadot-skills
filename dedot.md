# Dedot — Next-Gen Polkadot JavaScript/TypeScript Client

## Overview

Dedot is a delightful, tree-shakable JavaScript/TypeScript client for Polkadot and Substrate-based blockchains. It provides per-chain strongly-typed APIs via `@dedot/chaintypes`, supports Metadata V14/V15/V16, and works with both WebSocket and Smoldot light-client providers.

**Use Dedot when migrating from `@polkadot/api`** — its familiar syntax makes adoption straightforward. For brand-new projects, PAPI remains the default recommendation; Dedot is an excellent alternative with easier migration path from the legacy SDK.

Key differentiators:
- Familiar `@polkadot/api`-style syntax — minimal migration friction
- **Zero heavy dependencies** — no `bn.js`, no wasm blob
- Per-chain `ChainApi` interfaces via `@dedot/chaintypes` (autocomplete + compile-time safety)
- Unified smart contract APIs for ink! v5, v6, and Solidity/PVM
- Smoldot light-client support out of the box
- Compatible with `@polkadot/extension` wallets
- Web3 Foundation funded — [github.com/dedotdev/dedot](https://github.com/dedotdev/dedot)

## Installation

```bash
# Core package
npm i dedot

# Per-chain type suggestions (strongly recommended)
npm i -D @dedot/chaintypes
```

For Smoldot light-client support, also install chain specs:

```bash
npm i @dedot/chain-specs
```

## Connect to a Chain

### WebSocket Provider (development / server-side)

```typescript
import { DedotClient, WsProvider } from 'dedot';
import type { PolkadotApi } from '@dedot/chaintypes';

const provider = new WsProvider('wss://rpc.polkadot.io');
const client = await DedotClient.new<PolkadotApi>(provider);

console.log('Connected to Polkadot');

// Always disconnect when done
await client.disconnect();
```

### Smoldot Light Client (recommended for dApps)

```typescript
import { DedotClient, SmoldotProvider } from 'dedot';
import { start } from 'dedot/smoldot';
import { polkadot } from '@dedot/chain-specs';
import type { PolkadotApi } from '@dedot/chaintypes';

const smoldot = start();
const chain = await smoldot.addChain({ chainSpec: polkadot });
const provider = new SmoldotProvider(chain);
const client = await DedotClient.new<PolkadotApi>(provider);

console.log('Connected via Smoldot light client');
```

### Other Networks

```typescript
import type { KusamaApi, WestendApi, RococoApi } from '@dedot/chaintypes';

// Kusama
const ksm = await DedotClient.new<KusamaApi>(
  new WsProvider('wss://kusama-rpc.polkadot.io')
);

// Westend testnet
const wnd = await DedotClient.new<WestendApi>(
  new WsProvider('wss://westend-rpc.polkadot.io')
);
```

## Read Storage

```typescript
import { DedotClient, WsProvider } from 'dedot';
import type { PolkadotApi } from '@dedot/chaintypes';

const client = await DedotClient.new<PolkadotApi>(
  new WsProvider('wss://rpc.polkadot.io')
);

// Single storage query — fully typed return value
const accountInfo = await client.query.system.account('5GrwvaEF...');
console.log('Free balance:', accountInfo.data.free);

// Query multiple accounts at once
const balances = await client.query.system.account.multi([
  '5GrwvaEF...',
  '5FHneW46...',
]);
for (const [index, balance] of balances.entries()) {
  console.log(`Account ${index} free:`, balance.data.free);
}

// Read a constant
const existentialDeposit = await client.consts.balances.existentialDeposit;
console.log('Existential deposit:', existentialDeposit);

// Read other pallets
const totalIssuance = await client.query.balances.totalIssuance();
const validators = await client.query.session.validators();
```

## Subscribe to Storage Changes

```typescript
// Subscribe to account balance changes
const unsub = await client.query.system.account(
  '5GrwvaEF...',
  (accountInfo) => {
    console.log('Balance updated — free:', accountInfo.data.free);
  }
);

// Later, stop the subscription
await unsub();
```

## Submit Transactions

```typescript
import { DedotClient, WsProvider } from 'dedot';
import type { PolkadotApi } from '@dedot/chaintypes';

const client = await DedotClient.new<PolkadotApi>(
  new WsProvider('wss://rpc.polkadot.io')
);

// Sign and send — with status callback
const unsub = await client.tx.balances
  .transferKeepAlive('5FHneW46...', 2_000_000_000_000n)
  .signAndSend(alice, ({ status, events }) => {
    console.log('Transaction status:', status.type);

    if (status.type === 'BestChainBlockIncluded') {
      console.log('In best block:', status.value.blockHash);
    }

    if (status.type === 'Finalized') {
      console.log('Finalized in:', status.value.blockHash);
      unsub();
    }
  });

// Wait until finalized (async/await style)
const { status } = await client.tx.balances
  .transferKeepAlive('5FHneW46...', 2_000_000_000_000n)
  .signAndSend(alice, ({ status }) => {
    console.log('Status:', status.type);
  })
  .untilFinalized();

console.log('Finalized!', status);
```

## Signers

### Development Keypair (server-side only)

```typescript
import { DedotClient, WsProvider } from 'dedot';
import { Keyring } from '@polkadot/keyring';
import { cryptoWaitReady } from '@polkadot/util-crypto';

await cryptoWaitReady();
const keyring = new Keyring({ type: 'sr25519' });
const alice = keyring.addFromUri('//Alice');

// alice can now be passed directly to signAndSend
await client.tx.balances
  .transferKeepAlive('5FHneW46...', 1_000_000_000_000n)
  .signAndSend(alice);
```

### Browser Extension (client-side)

Dedot is compatible with any `@polkadot/extension`-compatible wallet (Talisman, SubWallet, Polkadot{.js} Extension):

```typescript
import { web3Accounts, web3Enable, web3FromAddress } from '@polkadot/extension-dapp';

// Enable extensions
const extensions = await web3Enable('My dApp');
if (!extensions.length) throw new Error('No extension found');

// Get accounts
const accounts = await web3Accounts();
const account = accounts[0];

// Get injector for the account
const injector = await web3FromAddress(account.address);

// Sign and send via extension
await client.tx.balances
  .transferKeepAlive('5FHneW46...', 1_000_000_000_000n)
  .signAndSend(account.address, { signer: injector.signer });
```

## Smart Contracts

Dedot provides unified contract APIs for ink! v5/v6 and Solidity (PVM). Use the [Typink](https://typink.dev) toolkit for a full dApp scaffolding experience.

```typescript
import { ContractDeployer, ContractClient } from 'dedot/contract';

// Deploy a contract
const deployer = new ContractDeployer(client, abi, wasm);
const { contractAddress } = await deployer
  .new(/* constructor args */)
  .signAndSend(alice)
  .untilFinalized();

// Interact with a deployed contract
const contract = new ContractClient(client, abi, contractAddress);

// Dry-run a query
const { data } = await contract.query.balanceOf(alice.address);
console.log('Token balance:', data);

// Send a transaction
await contract.tx
  .transfer('5FHneW46...', 1_000n)
  .signAndSend(alice);
```

## Best Practices

1. **Use Smoldot in production** — `SmoldotProvider` gives you a trustless light client without relying on centralized RPC nodes
2. **Always provide `ChainApi` type parameter** — `DedotClient.new<PolkadotApi>(provider)` unlocks full autocomplete and type safety
3. **Use `BigInt` for balances** — All Polkadot balances are `u128`, represented as `bigint` in JS/TS; never use plain numbers
4. **Unsubscribe when done** — Always call the returned `unsub()` function when you no longer need a storage subscription
5. **Use `.untilFinalized()`** — For transactions that need confirmation, `.untilFinalized()` gives clean async/await flow instead of manual status tracking
6. **Prefer `transferKeepAlive` over `transfer`** — Keeps the sender account alive (above existential deposit) to avoid account reaping
7. **Handle disconnections** — Wrap connection setup in try/catch and implement reconnect logic for production apps
8. **Tree-shake aggressively** — Dedot is built to be tree-shakable; import only what you need to minimize bundle size
