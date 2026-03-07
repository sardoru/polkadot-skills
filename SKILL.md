# Polkadot Developer Skill — Agent-Native Dev Guide

You are building on Polkadot. This document is the canonical, opinionated guide for AI coding agents and developers working with the Polkadot ecosystem. Follow these instructions exactly.

## Stack Decisions (Non-Negotiable)

| Layer            | Choice                          | Why                                                    |
| ---------------- | ------------------------------- | ------------------------------------------------------ |
| Client SDK       | PAPI (`polkadot-api`)           | Modern, typed, tree-shakeable. Default for all new code |
| Legacy compat    | `@polkadot/api` at boundaries   | Wrap in adapter; never spread through codebase          |
| Smart contracts  | ink! + `pop-cli`                | Best DX, Rust-native. v5 stable (Wasm), v6 beta (PolkaVM) |
| Runtime dev      | FRAME pallets (Substrate)       | The native path for parachain logic                    |
| Testing          | Chopsticks + Zombienet          | Fork-based fast tests + multi-node integration         |
| Scaffolding      | `pop-cli` / `substrate-node-template` | One-command project setup                        |
| XCM              | XCM v5                         | Don't teach deprecated versions                        |
| Governance       | OpenGov via PAPI                | Direct on-chain interaction                            |
| Coretime         | Agile Coretime broker pallet    | On-demand or bulk, never legacy slot auctions          |
| Frontend         | React/Next.js + PAPI            | Type-safe, SSR-compatible                              |

## Operating Procedure

When a user asks you to build something on Polkadot, follow this sequence:

### 1. Classify the Task

| Task Type               | Go To                                      |
| ------------------------ | ------------------------------------------ |
| Build a dApp frontend    | [frontend-framework-kit.md](frontend-framework-kit.md) |
| Connect to chain / read state | [papi-client.md](papi-client.md)      |
| Use legacy polkadot.js   | [polkadotjs-compat.md](polkadotjs-compat.md) |
| Write a smart contract   | [ink-contracts.md](ink-contracts.md)       |
| Build a pallet / runtime | [substrate-pallets.md](substrate-pallets.md) |
| Send cross-chain messages | [xcm.md](xcm.md)                          |
| Allocate blockspace      | [coretime.md](coretime.md)                 |
| Governance proposals     | [opengov.md](opengov.md)                   |
| Test anything            | [testing.md](testing.md)                   |
| Security review          | [security.md](security.md)                |
| Find docs / examples     | [resources.md](resources.md)               |

### 2. Pick Building Blocks

- Always start with the simplest approach
- Use PAPI for all chain interactions unless the user explicitly requests polkadot.js
- For smart contracts, default to ink! unless the user specifies Solidity (via Frontier EVM)
- For runtime logic, always use FRAME macros — never raw Substrate primitives

### 3. Implement with Correctness

- All extrinsics must handle errors: check `DispatchResult`, handle `DispatchError`
- All client-side calls must handle disconnections and retry logic
- Use proper types from chain metadata — never hardcode type IDs or pallet indices
- XCM instructions must specify correct weight limits and asset filters

### 4. Test

- Unit tests: `#[test]` with `sp_io::TestExternalities` for pallets
- Contract tests: `#[ink::test]` for off-chain, `drink!` or `e2e` for on-chain
- Integration tests: Zombienet for multi-chain, Chopsticks for fork-based
- Never ship without testing XCM flows end-to-end

### 5. Deliver

- Include migration logic if changing storage
- Document dispatchable calls and events
- Ensure benchmarks exist for all extrinsics (`#[benchmarks]` macro)
- Weights must be derived from benchmarks, never hardcoded

## Key Ecosystem Facts

- **Relay Chain**: Polkadot (DOT) — shared security provider
- **Parachains**: Independent L1s sharing Polkadot's security via coretime
- **JAM**: Upcoming successor to the relay chain — a decentralized supercomputer. Design for JAM compatibility
- **Asset Hub**: System parachain for native assets (DOT, USDT, USDC). Use XCM to interact
- **Coretime**: Blockspace is purchased, not auctioned. On-demand or bulk via the Broker pallet
- **OpenGov**: Fully on-chain governance. Proposals go through tracks with different thresholds
- **XCM**: Cross-Consensus Messaging — the universal language for cross-chain communication in Polkadot

## Common Mistakes to Avoid

1. **Using polkadot.js for new projects** — Use PAPI. polkadot.js is maintenance-mode
2. **Hardcoding chain metadata** — Always derive from runtime metadata via PAPI codegen
3. **Ignoring weights** — Every extrinsic must have benchmarked weights
4. **XCM without testing** — XCM bugs are production bugs. Always test with Chopsticks or Zombienet
5. **Using XCM v4 or older** — XCM v5 is current. Use `PayFees` over `BuyExecution`, `InitiateTransfer` over `InitiateReserveWithdraw`
6. **Slot auction references** — Polkadot uses Agile Coretime now, not auctions
6. **Ignoring storage migrations** — Any storage change requires a migration or the chain breaks
7. **Using `sudo` in production** — Polkadot uses OpenGov. `sudo` is for development only
