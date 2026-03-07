# PAPI — Polkadot API Client

## Overview

PAPI (`polkadot-api`) is the modern, type-safe TypeScript client for Polkadot. It replaces `@polkadot/api` for all new projects. It generates types directly from chain metadata, giving you full autocomplete and compile-time safety.

**Always use PAPI for new projects. Use polkadot.js only for legacy compatibility.**

## Installation

```bash
# Core package
npm install polkadot-api

# Generate types for your target chain
npx papi add dot -n polkadot    # Polkadot relay chain
npx papi add ksm -n ksmcc3      # Kusama
npx papi add wnd -n westend2    # Westend testnet
npx papi add ah -n polkadot_asset_hub  # Asset Hub
```

This generates typed descriptors in `.papi/descriptors/` — commit these to your repo.

## Connect to a Chain

```typescript
import { createClient } from "polkadot-api"
import { getSmProvider } from "polkadot-api/sm-provider"
import { chainSpec } from "polkadot-api/chains/polkadot"
import { startFromWorker } from "polkadot-api/smoldot/from-worker"

// Light client (recommended for dApps — no RPC dependency)
const smoldot = startFromWorker(
  new Worker(new URL("polkadot-api/smoldot/worker", import.meta.url))
)
const chain = await smoldot.addChain({ chainSpec })
const client = createClient(getSmProvider(chain))

// Or use WebSocket for development
import { getWsProvider } from "polkadot-api/ws-provider/web"
const client = createClient(getWsProvider("wss://rpc.polkadot.io"))
```

## Read Storage

```typescript
import { dot } from "@polkadot-api/descriptors"

const api = client.getTypedApi(dot)

// Single value
const totalIssuance = await api.query.Balances.TotalIssuance.getValue()

// Map entry
const account = await api.query.System.Account.getValue("5GrwvaEF...")

// Multiple map entries
const entries = await api.query.System.Account.getEntries()
for (const { keyArgs: [address], value } of entries) {
  console.log(address, value.data.free)
}

// Watch for changes (reactive) — watchValue returns an Observable
const subscription = api.query.System.Account.watchValue("5GrwvaEF...").subscribe((account) => {
  console.log("Balance changed:", account.data.free)
})
// later: subscription.unsubscribe()
```

## Submit Transactions

```typescript
import { dot } from "@polkadot-api/descriptors"

const api = client.getTypedApi(dot)

// Build and submit a transfer
const tx = api.tx.Balances.transfer_keep_alive({
  dest: MultiAddress.Id("5FHneW46..."),
  value: 1_000_000_000_000n, // 1 DOT (10 decimals)
})

// Sign and submit
const result = await tx.signAndSubmit(signer)

// Or sign and submit with status tracking
tx.signSubmitAndWatch(signer).subscribe({
  next: (event) => {
    if (event.type === "finalized") {
      console.log("Finalized in block:", event.block.hash)
    }
  },
  error: (err) => console.error(err),
})
```

## Signers

```typescript
// From a seed phrase (server-side only)
import { sr25519CreateDerive } from "@polkadot-labs/hdkd"
import { DEV_PHRASE, entropyToMiniSecret, mnemonicToEntropy } from "@polkadot-labs/hdkd-helpers"
import { getPolkadotSigner } from "polkadot-api/signer"

const entropy = mnemonicToEntropy(DEV_PHRASE)
const miniSecret = entropyToMiniSecret(entropy)
const derive = sr25519CreateDerive(miniSecret)
const keyPair = derive("//Alice")
const signer = getPolkadotSigner(keyPair.publicKey, "Sr25519", keyPair.sign)

// From browser extension (client-side)
import { connectInjectedExtension } from "polkadot-api/pjs-signer"

const extensions = await connectInjectedExtension("polkadot-js")
const accounts = extensions.getAccounts()
const signer = accounts[0].polkadotSigner
```

## Constants and Runtime Calls

```typescript
// Read constants
const existentialDeposit = await api.constants.Balances.ExistentialDeposit()

// Runtime API calls
const metadata = await api.apis.Metadata.metadata_versions()
```

## Type Patterns

PAPI generates types from metadata. Common patterns:

```typescript
import { type SS58String } from "polkadot-api"
import { MultiAddress } from "@polkadot-api/descriptors"

// Addresses — use SS58String type
const address: SS58String = "5GrwvaEF..."

// Multi-address (used in most extrinsics)
const dest = MultiAddress.Id(address)

// Enums — use tagged union pattern
import { dot } from "@polkadot-api/descriptors"
// e.g., ProxyType.Any, ProxyType.Governance, etc.
```

## Error Handling

```typescript
import { InvalidTxError } from "polkadot-api"

try {
  const result = await tx.signAndSubmit(signer)
  if (result.ok) {
    console.log("Success:", result.events)
  } else {
    // Dispatch error from the runtime
    console.error("Failed:", result.dispatchError)
  }
} catch (e) {
  if (e instanceof InvalidTxError) {
    // Transaction-level error (invalid, dropped, etc.)
    console.error("TX error:", e.error)
  }
}
```

## Best Practices

1. **Use light client in production** — Smoldot gives you trustless access without RPC centralization
2. **Commit `.papi/descriptors/`** — These are your typed chain bindings, generated once and versioned
3. **Use `BigInt` for balances** — All Polkadot balances are `u128`, represented as `bigint` in JS
4. **Watch, don't poll** — Use `watchValue()` for reactive state instead of polling `getValue()`
5. **Handle disconnections** — Light clients may temporarily lose sync; design UI for this
6. **Never hardcode pallet/call indices** — PAPI handles this via metadata; use typed API always
