# polkadot.js Compatibility Layer

## Status: Maintenance Mode

`@polkadot/api` is in maintenance mode. **Do not use it for new projects.** Use [PAPI](papi-client.md) instead.

Use polkadot.js only when:
- Integrating with existing codebases that already use it
- Using libraries that require `@polkadot/api` types (some DeFi SDKs)
- Interacting with tools that haven't migrated yet

## Compatibility Pattern

When you must use polkadot.js, isolate it at boundaries. Never let polkadot.js types leak into your core application logic.

```typescript
// adapters/polkadotjs.ts — Boundary layer
import { ApiPromise, WsProvider } from "@polkadot/api"
import type { AccountInfo } from "your-app/types"

export async function getAccountInfo(address: string): Promise<AccountInfo> {
  const provider = new WsProvider("wss://rpc.polkadot.io")
  const api = await ApiPromise.create({ provider })

  const account = await api.query.system.account(address)

  // Convert to your app's types at the boundary
  return {
    free: account.data.free.toBigInt(),
    reserved: account.data.reserved.toBigInt(),
    nonce: account.nonce.toNumber(),
  }
}
```

## Quick Reference (polkadot.js)

### Connect

```typescript
import { ApiPromise, WsProvider } from "@polkadot/api"

const provider = new WsProvider("wss://rpc.polkadot.io")
const api = await ApiPromise.create({ provider })
```

### Read Storage

```typescript
// Single value
const totalIssuance = await api.query.balances.totalIssuance()

// Map entry
const account = await api.query.system.account("5GrwvaEF...")

// Subscribe
const unsub = await api.query.system.account("5GrwvaEF...", (account) => {
  console.log("Balance:", account.data.free.toHuman())
})
```

### Submit Transaction

```typescript
import { Keyring } from "@polkadot/keyring"

const keyring = new Keyring({ type: "sr25519" })
const alice = keyring.addFromUri("//Alice")

const hash = await api.tx.balances
  .transferKeepAlive("5FHneW46...", 1_000_000_000_000n)
  .signAndSend(alice)
```

## Migration Checklist (polkadot.js -> PAPI)

| polkadot.js | PAPI |
|-------------|------|
| `ApiPromise.create()` | `createClient()` + `getTypedApi()` |
| `api.query.system.account()` | `api.query.System.Account.getValue()` |
| `api.tx.balances.transfer()` | `api.tx.Balances.transfer_keep_alive()` |
| `WsProvider` | `getWsProvider()` or Smoldot |
| `Keyring` | `getPolkadotSigner()` with hdkd |
| `.toBigInt()` / `.toNumber()` | Native bigint / number (already typed) |
| `api.consts.balances.existentialDeposit` | `api.constants.Balances.ExistentialDeposit()` |

Key differences:
- PAPI uses **PascalCase** for pallet names: `Balances`, `System` (not `balances`, `system`)
- PAPI returns **native JS types** — no `.toHuman()`, `.toBigInt()` wrappers
- PAPI generates types from metadata — full autocomplete, no `any`
