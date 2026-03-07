# Polkadot Treasury Proposal: Polkadot Agent Mesh

## Proposal Summary

| Field | Value |
|-------|-------|
| **Title** | Polkadot Agent Mesh: Agent-Native Developer Skill + Agent Spaces Infrastructure |
| **Proposer** | [Your Name / Entity] |
| **Beneficiary Address** | [Polkadot SS58 address with verified on-chain identity] |
| **Requested Amount (Phase 1)** | 1,430 DOT (~$10,000 at $7/DOT) |
| **Track** | Small Spender |
| **Category** | Developer Tooling / Documentation / Infrastructure |

---

## Abstract

This proposal funds the deployment and maintenance of **Polkadot SKILL.md** — a canonical, agent-native developer guide that enables AI coding agents (Claude Code, Cursor, Codex, Copilot) to build correctly on Polkadot — and the specification of **Agent Spaces**, a coretime-powered infrastructure for agent-to-agent economies.

---

## Problem Statement

### 1. AI Agents Cannot Build on Polkadot Correctly

In 2026, the majority of new code is written with AI assistance. These AI agents rely on structured, machine-readable documentation to generate correct code. Solana has shipped `solana.com/SKILL.md` — a canonical guide that any AI agent can ingest. Polkadot has no equivalent.

**Current failure modes when AI agents target Polkadot:**
- Recommend `@polkadot/api` (maintenance-mode) instead of PAPI
- Reference slot auctions (deprecated since Agile Coretime)
- Generate XCM v3 code (v5 is current)
- Hallucinate npm package names that don't exist
- Miss critical tools: `pop-cli`, Chopsticks, Zombienet

Every AI-assisted developer who receives broken Polkadot code is a developer lost to a competing ecosystem.

### 2. The AI Agent Infrastructure Race

Ethereum has announced ERC-8004 — their strategy to become the "trust layer for AI agents." Polkadot has superior architecture for this use case (isolated execution domains, XCM, coretime, OpenGov, Proof-of-Personhood) but no one is building the agent-native infrastructure.

---

## Solution

### Deliverable 1: Polkadot SKILL.md (Completed, Needs Deployment)

A complete, validated, opinionated developer guide comprising 12 documents and 2,270 lines covering:

- **SKILL.md** — Main entry with stack decisions and operating procedure
- **substrate-pallets.md** — FRAME pallet development with `#[frame::pallet]`, `VersionedMigration`, benchmarking
- **ink-contracts.md** — ink! v5 smart contracts with v6/PolkaVM transition notes
- **papi-client.md** — PAPI client SDK with Smoldot light client, Observable patterns
- **polkadotjs-compat.md** — Legacy compatibility layer (boundary pattern)
- **xcm.md** — XCM v5 with `PayFees`, `InitiateTransfer`, `SetHints`
- **coretime.md** — Agile Coretime with Broker pallet operations
- **opengov.md** — All 15 governance tracks with correct spend limits and dynamic curves
- **testing.md** — Chopsticks (`@acala-network/chopsticks`), Zombienet (`@zombienet/cli`), try-runtime
- **security.md** — Substrate + ink! + XCM security checklist including v5 Empowered Origins
- **frontend-framework-kit.md** — React/Next.js + PAPI patterns
- **resources.md** — Curated links with correct repository paths

**Validation:** 6 parallel research agents cross-referenced every document against live sources. 44 corrections applied across 10 files.

### Deliverable 2: Agent Spaces Specification (Phase 2)

Design specification for coretime-powered agent execution environments:

- **PRC-8004** — Polkadot-native agent identity standard (inspired by Ethereum's ERC-8004)
- **Agent Registry Chain** — System parachain for identity, reputation, and validation registries
- **Agent XCM Protocol** — Extended XCM for agent-to-agent messages
- **Reference Agent Spaces** — DeFi, Governance, and Creative verticals

---

## Milestones

### Phase 1: SKILL.md Deployment + Maintenance (This Proposal)

| # | Milestone | Deliverable | Cost | Timeline |
|---|-----------|-------------|------|----------|
| 1.1 | Technical Review | Feedback incorporated from Parity/W3F engineers and active Polkadot devs | $2,000 | Week 1-2 |
| 1.2 | Deployment | SKILL.md hosted on official Polkadot docs or website; PR merged | $1,000 | Week 2-3 |
| 1.3 | Distribution | Registered on ClawhHub, OpenClaw, and agent skill directories | $500 | Week 3 |
| 1.4 | Maintenance | 3 monthly update cycles tracking PAPI releases, XCM changes, ink! v6 progress | $4,500 | Month 1-3 |
| 1.5 | Amplification | Forum posts, Twitter campaign, Polkadot blog article, dev community outreach | $2,000 | Month 1 |
| | **Total** | | **$10,000** | **3 months** |

**Success metrics:**
- SKILL.md merged into official Polkadot documentation
- Listed on at least 3 AI agent skill directories
- 50+ GitHub stars on the repository
- Measurable reduction in "polkadot.js" vs "PAPI" usage in AI-generated code (tracked via community surveys)

### Phase 2: Agent Spaces Specification (Future Proposal)

| # | Milestone | Cost | Timeline |
|---|-----------|------|----------|
| 2.1 | PRC-8004 specification | $8,000 | Month 1-2 |
| 2.2 | Agent XCM Protocol RFC | $6,000 | Month 2-3 |
| 2.3 | Agent Registry pallet (reference impl) | $12,000 | Month 2-3 |
| 2.4 | Reference Agent Space runtime | $10,000 | Month 3 |
| 2.5 | Documentation + developer guide | $4,000 | Month 3 |
| | **Total** | **$40,000** | **3 months** |

### Phase 3: Production Launch (Future Bounty Proposal)

| # | Milestone | Cost | Timeline |
|---|-----------|------|----------|
| 3.1 | Agent Registry Chain (testnet) | $20,000 | Month 1-3 |
| 3.2 | 3 reference Agent Spaces | $30,000 | Month 2-5 |
| 3.3 | Agent XCM live parachain integration | $15,000 | Month 3-6 |
| 3.4 | Security audit | $25,000 | Month 5-6 |
| 3.5 | JAM compatibility + migration path | $10,000 | Month 4-6 |
| | **Total** | **$100,000** | **6 months** |

---

## Budget Justification

Phase 1 is deliberately scoped as a Small Spender proposal ($10,000 / ~1,430 DOT) to minimize governance friction and demonstrate value before requesting larger allocations.

| Item | Rate | Hours | Cost |
|------|------|-------|------|
| Technical review + revision cycles | $100/hr | 20 | $2,000 |
| Deployment + CI/CD for maintenance | $100/hr | 10 | $1,000 |
| Skill directory registration + docs | $100/hr | 5 | $500 |
| Monthly SDK tracking + updates (3 mo) | $100/hr | 45 | $4,500 |
| Community outreach + content creation | $100/hr | 20 | $2,000 |
| **Total** | | **100 hrs** | **$10,000** |

---

## Why Polkadot Is Uniquely Positioned

| Capability | Ethereum | Polkadot |
|-----------|----------|----------|
| Isolated agent execution domains | No (L2 fragmentation) | Yes (Parachains / Coretime) |
| Custom rules per agent domain | No (one EVM) | Yes (per-parachain runtime) |
| Native cross-domain agent messaging | No (bridges) | Yes (XCM) |
| On-chain agent governance | No (off-chain) | Yes (OpenGov) |
| Flexible compute allocation | No | Yes (Agile Coretime) |
| Proof-of-Personhood | No | Yes (active research) |

---

## Team

[To be completed]

| Name | Role | Background |
|------|------|-----------|
| [Lead] | Project Lead | [Experience with Polkadot, Substrate, or relevant blockchain development] |
| [Dev 1] | Substrate Developer | [Pallet development, XCM integration experience] |
| [Dev 2] | Frontend / SDK | [PAPI, TypeScript, dApp development experience] |
| [Advisor] | Technical Advisor | [Polkadot ecosystem contributor, governance participant] |

---

## Risk Assessment

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| SKILL.md becomes outdated | Medium | Monthly maintenance budget; CI checks against PAPI/XCM releases |
| Low adoption by AI agents | Low | Register on all major skill directories; Solana has proven the model works |
| ink! v6 breaks contract guidance | Medium | v5/v6 dual guidance already included; monthly tracking |
| Agent Spaces concept doesn't gain traction | Medium | Phase 1 (SKILL.md) delivers standalone value regardless |
| Competing SKILL.md efforts | Low | First-mover advantage; propose for official adoption |

---

## Prior Art

- **Solana SKILL.md** — Proven model. Solana shipped an agent-native dev guide and saw immediate adoption by AI coding tools.
- **Ethereum ERC-8004** — Agent identity standard. Our PRC-8004 is inspired by this but designed for Polkadot's multi-chain architecture.
- **Polkadot Developer Heroes** — Community education program. Our SKILL.md complements this by targeting AI agents, not just human developers.
- **DotCodeSchool** — Interactive learning platform. Our work serves a different audience (AI agents) but shares the goal of correct Polkadot development.

---

## Appendix: Repository Structure

```
polkadot-agent-mesh/
|- PROPOSAL.md                  <- Agent Spaces architecture proposal
|- SKILL.md                     <- Main entry (stack decisions + operating procedure)
|- substrate-pallets.md         <- FRAME pallet development
|- ink-contracts.md             <- ink! smart contracts
|- papi-client.md               <- PAPI client SDK
|- polkadotjs-compat.md         <- Legacy compatibility layer
|- xcm.md                       <- Cross-chain messaging (XCM v5)
|- coretime.md                  <- Agile Coretime
|- opengov.md                   <- Governance integration
|- testing.md                   <- Testing tools
|- security.md                  <- Security checklist
|- frontend-framework-kit.md    <- Frontend patterns
|- resources.md                 <- Curated links
|- campaign/
   |- TREASURY_PROPOSAL.md      <- This document
   |- FORUM_POST.md             <- Polkadot Forum post
   |- TWITTER_THREAD.md         <- Social media campaign
   |- CALL_TO_ACTION.md         <- Community rally document
   |- BOUNTY_PROPOSAL.md        <- Phase 3 bounty structure
```
