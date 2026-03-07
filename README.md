# Polkadot Agent Mesh

**The agent-native developer guide for Polkadot.** A canonical, validated set of instructions that any AI coding agent — Claude Code, Cursor, Codex, Copilot — can ingest to build correctly on Polkadot.

Solana shipped `solana.com/SKILL.md`. This is Polkadot's answer — and then some.

---

## Why This Exists

AI agents writing Polkadot code today are broken. They recommend `@polkadot/api` (maintenance-mode), reference slot auctions (deprecated since Agile Coretime), generate XCM v3 patterns (v5 is current), and hallucinate npm packages that don't exist. Every broken build is a developer lost to a competing ecosystem.

This repository fixes that with two things:

1. **SKILL.md** — A complete, opinionated developer guide that tells AI agents exactly how to build on Polkadot
2. **Agent Spaces** — A proposal for coretime-powered blockspaces purpose-built for agent-to-agent economies

---

## Quick Start

### For AI Coding Agents

Point your agent at [`SKILL.md`](SKILL.md). It contains the full operating procedure: classify the task, pick the right building blocks, implement with correctness, test, and deliver.

### For Developers

Read the sub-document that matches your task:

| I want to... | Read this |
|--------------|-----------|
| Build a dApp frontend | [frontend-framework-kit.md](frontend-framework-kit.md) |
| Connect to a chain / read state | [papi-client.md](papi-client.md) |
| Use legacy polkadot.js | [polkadotjs-compat.md](polkadotjs-compat.md) |
| Write a smart contract | [ink-contracts.md](ink-contracts.md) |
| Build a pallet / runtime | [substrate-pallets.md](substrate-pallets.md) |
| Send cross-chain messages | [xcm.md](xcm.md) |
| Allocate blockspace | [coretime.md](coretime.md) |
| Submit governance proposals | [opengov.md](opengov.md) |
| Test anything | [testing.md](testing.md) |
| Review security | [security.md](security.md) |
| Find docs / examples | [resources.md](resources.md) |

---

## Stack Decisions

These are non-negotiable. Every document in this repo follows these choices:

| Layer | Choice | Why |
|-------|--------|-----|
| Client SDK | **PAPI** (`polkadot-api`) | Modern, typed, tree-shakeable. Default for all new code |
| Legacy compat | `@polkadot/api` at boundaries only | Wrap in adapter; never spread through codebase |
| Smart contracts | **ink!** + `pop-cli` | Best DX, Rust-native. v5 stable (Wasm), v6 beta (PolkaVM) |
| Runtime dev | **FRAME** pallets (`#[frame::pallet]`) | Unified `frame` crate, the native path |
| Testing | **Chopsticks** + **Zombienet** | Fork-based fast tests + multi-node integration |
| XCM | **v5 only** | `PayFees` over `BuyExecution`, `InitiateTransfer` over `InitiateReserveWithdraw` |
| Governance | **OpenGov** via PAPI | Direct on-chain interaction, all 15 tracks |
| Coretime | **Agile Coretime** (Broker pallet) | On-demand or bulk. Never reference slot auctions |
| Frontend | **React/Next.js** + PAPI + Smoldot | Type-safe, SSR-compatible, trustless light client |

---

## Repository Structure

```
polkadot-agent-mesh/
|
|-- Core Developer Guide (SKILL.md)
|   |-- SKILL.md                     Main entry point (stack decisions + operating procedure)
|   |-- substrate-pallets.md         FRAME pallet dev (#[frame::pallet], VersionedMigration, benchmarks)
|   |-- ink-contracts.md             ink! smart contracts (v5 stable, v6/PolkaVM transition)
|   |-- papi-client.md               PAPI client SDK (Smoldot, Observable queries, signers)
|   |-- polkadotjs-compat.md         Legacy polkadot.js compatibility (boundary pattern)
|   |-- xcm.md                       XCM v5 (PayFees, InitiateTransfer, SetHints, locations)
|   |-- coretime.md                  Agile Coretime (Broker.partition, on-demand ordering, bulk)
|   |-- opengov.md                   OpenGov (15 tracks, dynamic curves, conviction voting)
|   |-- testing.md                   Chopsticks, Zombienet, try-runtime
|   |-- security.md                  Substrate + ink! + XCM security checklist
|   |-- frontend-framework-kit.md    React/Next.js + PAPI patterns
|   |-- resources.md                 Curated links, RPCs, repos, learning path
|
|-- Agent Spaces Proposal
|   |-- PROPOSAL.md                  Architecture for coretime-powered agent economies
|
|-- Campaign Materials
|   |-- campaign/FORUM_POST.md       Polkadot Forum discussion post
|   |-- campaign/TREASURY_PROPOSAL.md Formal treasury proposal (Phase 1: Small Spender)
|   |-- campaign/BOUNTY_PROPOSAL.md  Phase 3 bounty structure (18 child bounties)
|   |-- campaign/CALL_TO_ACTION.md   Community rally document
|   |-- campaign/TWITTER_THREAD.md   12-tweet social media campaign
|
|-- README.md                        This file
```

---

## Validation

Every document was validated against live sources by 6 parallel research agents. Here's what was checked and corrected:

| Document | Validated Against | Corrections |
|----------|-------------------|-------------|
| papi-client.md | papi.how, npm registry, polkadot-api repo | `watchValue` Observable pattern, `InvalidTxError`, `MultiAddress` import, chain names |
| ink-contracts.md | use.ink, ink! repo, pop-cli docs | Added v5/v6 status, unified `pop build`/`pop up`, `pendzl` only |
| xcm.md | Polkadot SDK, XCM v5 docs, Fellowship RFCs | v4 to v5 throughout, `PayFees`/`InitiateTransfer`/`SetHints`, correct package names |
| substrate-pallets.md | polkadot-sdk repo, FRAME docs | `#[frame::pallet]`, `VersionedMigration`, `dev_mode` |
| coretime.md | Polkadot wiki, Broker pallet source | `Broker.partition` (not split), ~18-25 cores (not ~50), explicit on-demand ordering |
| opengov.md | Polkadot wiki, governance docs | All 6 treasury spend limits corrected, dynamic curves (not majority), added missing tracks |
| testing.md | Chopsticks repo, Zombienet repo | `@acala-network/chopsticks`, `@zombienet/cli`, no `dev` subcommand, `paseo-local` |
| security.md | Polkadot security guides | Added XCM v5 Empowered Origins, Snowbridge V2 |
| resources.md | GitHub, npm | All repo paths corrected (`paritytech/`, `use-ink/`, `AcalaNetwork/`) |

**44 total corrections** applied across 10 files. Zero hallucinated package names or deprecated APIs remain.

---

## Agent Spaces Proposal

Beyond the developer guide, this repo contains a design for **Agent Spaces** — dedicated coretime-powered blockspaces for agent-to-agent economies.

### Three Tiers

**1. Agent Registry Chain** (system parachain)
- PRC-8004: Polkadot-native agent identity, reputation, and validation registries
- Proof-of-Personhood integration (human-controlled vs autonomous agents)
- XCM-accessible from any parachain

**2. Agent Spaces** (coretime-powered)
- Any DAO, community, or enterprise can spin up an Agent Space via coretime
- Custom rules per vertical: who can register, fee structures, dispute resolution
- Examples: DeFi agents (circuit breakers, slashing), Governance agents (identity-verified, human-override), Supply Chain agents (KYB-verified, SLA enforcement)

**3. Agent XCM Protocol**
- Extended XCM for agent-to-agent messages
- Task delegation, payment routing, reputation queries across spaces
- Escrow-based payments released on task validation

### Why Polkadot (Not Ethereum)

Ethereum's ERC-8004 puts all agents on one congested chain. Polkadot gives each agent community its own execution environment with custom rules, native interop, and shared security.

> "Ethereum gives agents a shared apartment. Polkadot gives them a city — with neighborhoods, local laws, and a transit system."

Read the full proposal: [PROPOSAL.md](PROPOSAL.md)

---

## Treasury Funding

This project is seeking Polkadot Treasury funding in three phases:

| Phase | Track | Amount | Status |
|-------|-------|--------|--------|
| 1. SKILL.md deployment + maintenance | Small Spender | ~$10,000 (1,430 DOT) | Preparing |
| 2. Agent Spaces specification + reference impl | Medium Spender | ~$40,000 | Planned |
| 3. Production launch (bounty) | Big Spender | ~$100,000 | Planned |

See [campaign/TREASURY_PROPOSAL.md](campaign/TREASURY_PROPOSAL.md) for the full Phase 1 proposal.

See [campaign/BOUNTY_PROPOSAL.md](campaign/BOUNTY_PROPOSAL.md) for the Phase 3 bounty structure with 18 child bounties open to any contributor.

---

## How to Contribute

### Review the Docs

Read the SKILL.md and sub-documents. Open an issue if you find:
- API calls that don't match current SDK versions
- Patterns you'd never use in production
- Missing security considerations
- Incorrect extrinsic names or parameter types

### Test with Your AI Agent

Drop `SKILL.md` into your Claude Code, Cursor, or Copilot context. Ask it to build something on Polkadot. Report whether it generates correct code.

### Add Parachain-Specific Guidance

Want AI agents to build correctly on your parachain? Submit a PR with a chain-specific sub-document (e.g., `astar-contracts.md`, `moonbeam-evm.md`).

### Support the Treasury Proposal

- Comment on the [Polkadot Forum post](#) with feedback or endorsement
- Vote Aye when the referendum goes live
- Share on Twitter/X with `#PolkadotAgentMesh`

---

## Links

| Resource | URL |
|----------|-----|
| Polkadot Wiki | https://wiki.polkadot.network |
| Polkadot Developer Docs | https://docs.polkadot.com |
| PAPI Docs | https://papi.how |
| ink! Docs | https://use.ink |
| Polkadot Forum | https://forum.polkadot.network |
| Polkassembly | https://polkadot.polkassembly.io |
| Subsquare | https://polkadot.subsquare.io |

---

## License

Open source. This documentation is intended to be freely used, shared, and contributed to by the Polkadot community.
