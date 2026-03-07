# Polkadot Agent Mesh — Project Instructions

## What This Is

Agent-native developer guide for Polkadot. 12 validated documents covering the full dev stack. Plus an Agent Spaces architecture proposal.

## Stack Decisions (Non-Negotiable)

| Layer | Use | NOT |
|-------|-----|-----|
| Client SDK | PAPI (`polkadot-api`) | polkadot.js (`@polkadot/api`) |
| Smart Contracts | ink! + `pop-cli` | Solidity, Hardhat |
| Runtime | FRAME with unified `frame` crate | `frame_support` directly |
| Cross-Chain | XCM v5 | XCM v3/v4 |
| Testing | Chopsticks + Zombienet | Substrate `--dev` node only |
| Governance | OpenGov (15 tracks, dynamic curves) | Gov1 / static majority |
| Blockspace | Agile Coretime (Broker pallet) | Slot auctions |
| Scaffolding | `pop-cli` | `substrate-node-template` |

## Key Patterns

- **PAPI chain names**: `polkadot`, `ksmcc3` (not `kusama`), `westend2` (not `westend`), `polkadot_asset_hub`
- **PAPI Observable**: `watchValue` returns an Observable — use `.subscribe()`, NOT a callback
- **MultiAddress**: Import from `@polkadot-api/descriptors`, not `polkadot-api`
- **FRAME macro**: `#[frame::pallet]` with `frame::prelude::*`
- **Migrations**: `VersionedMigration` + `UncheckedOnRuntimeUpgrade` (not bare `OnRuntimeUpgrade`)
- **XCM v5 fees**: `PayFees` (not `BuyExecution`)
- **XCM v5 transfers**: `InitiateTransfer` (not `InitiateReserveWithdraw`/`InitiateTeleport`)
- **Coretime split**: `Broker.partition` (not `Broker.split`)
- **ink! build**: `pop build` / `pop up` (unified, not `pop build contract`)
- **Test network**: `paseo-local` (not `rococo-local`)
- **Error class**: `InvalidTxError` (not `TransactionError`)

## File Structure

```
SKILL.md              — Main entry point for AI agents
papi-client.md        — PAPI SDK guide
substrate-pallets.md  — FRAME pallet development
ink-contracts.md      — ink! smart contracts
xcm.md                — XCM v5 cross-chain messaging
coretime.md           — Agile Coretime / Broker pallet
opengov.md            — OpenGov governance
testing.md            — Chopsticks, Zombienet, try-runtime
security.md           — Security checklist
frontend-framework-kit.md — React/Next.js + PAPI
polkadotjs-compat.md  — Legacy compatibility layer
resources.md          — Curated links and tools
PROPOSAL.md           — Agent Spaces architecture
```

## When Editing Documents

- All code examples must use PAPI, never polkadot.js (except in polkadotjs-compat.md boundary examples)
- Validate API names against actual npm packages and GitHub repos before writing
- Use `xcm::latest::prelude::*` for XCM code, never pin to a version number
- Keep the llms-full.txt file in sync — it's a concatenation of all 13 core docs
- Update .well-known/agent.json if adding new documents

## Common Hallucination Traps

These are things AI agents frequently get wrong about Polkadot:

1. `@nicepicks` scope — does not exist. Real packages: `@acala-network/chopsticks`, `@zombienet/cli`
2. `Broker.split` — the extrinsic is `Broker.partition`
3. `watchValue(callback)` — PAPI returns Observable, use `.subscribe()`
4. `docs.substrate.io` — moved to `docs.polkadot.com`
5. ~50 Polkadot cores — actual count is ~18-25
6. On-demand coretime is automatic — it requires explicit `placeOrderAllowDeath`/`placeOrderKeepAlive`
7. OpenGov uses simple/super majority — it uses dynamic approval/support curves
8. Chopsticks `dev` subcommand — doesn't exist, use `chopsticks --config=`
