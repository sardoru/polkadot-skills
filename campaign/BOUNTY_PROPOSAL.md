# Polkadot Agent Mesh — Bounty Proposal (Phase 3)

## Bounty Title

**Polkadot Agent Mesh: Agent-Native Infrastructure for Polkadot**

## Bounty Description

This bounty funds the production deployment of Polkadot's agent-native infrastructure stack — from the SKILL.md developer guide through to live Agent Spaces running on coretime. The bounty structure enables phased funding through child bounties without requiring separate referenda for each milestone.

## Motivation

The AI agent economy is the next major platform shift. Ethereum (ERC-8004), Solana (SKILL.md), and NEAR (agent frameworks) are all positioning for it. Polkadot has the best architecture for multi-agent systems (isolated execution domains, XCM, coretime, OpenGov, Proof-of-Personhood) but zero agent-native infrastructure.

This bounty fills that gap across three dimensions:

1. **Developer layer** — Make AI agents build correctly on Polkadot (SKILL.md)
2. **Identity layer** — Give agents on-chain identity, reputation, and validation (PRC-8004)
3. **Execution layer** — Give agent economies dedicated blockspace with custom rules (Agent Spaces)

## Bounty Value

**Total: 14,285 DOT (~$100,000 at $7/DOT)**

This is proposed as a **Big Spender** track bounty with child bounties for individual deliverables.

## Curator

[Proposed curator(s) — should be recognized Polkadot community members with technical credibility]

- **Curator Fee:** 5% (714 DOT)
- **Curator Deposit:** Per protocol requirements

## Child Bounty Structure

### Track A: Developer Tooling (3,570 DOT / ~$25,000)

| Child Bounty | Deliverable | DOT | Status |
|-------------|-------------|-----|--------|
| A1 | SKILL.md deployment to official Polkadot docs | 285 | Ready |
| A2 | Agent skill directory registration (ClawhHub, OpenClaw) | 145 | Ready |
| A3 | 6-month maintenance (monthly SDK tracking + updates) | 1,285 | Ongoing |
| A4 | Parachain-specific extensions (Astar, Moonbeam, Acala) | 1,000 | Open call |
| A5 | Community education (blog posts, video walkthroughs) | 570 | Open call |
| A6 | Localization (Chinese, Japanese, Korean, Spanish) | 285 | Open call |

### Track B: Agent Identity Standard (4,285 DOT / ~$30,000)

| Child Bounty | Deliverable | DOT | Status |
|-------------|-------------|-----|--------|
| B1 | PRC-8004 specification (RFC) | 1,145 | Spec phase |
| B2 | Agent Registry pallet (Substrate implementation) | 1,715 | Dev phase |
| B3 | Agent Registry PAPI client library | 570 | Dev phase |
| B4 | Proof-of-Personhood integration module | 570 | Research phase |
| B5 | Agent Registry testnet deployment | 285 | Deployment |

### Track C: Agent Spaces Infrastructure (6,430 DOT / ~$45,000)

| Child Bounty | Deliverable | DOT | Status |
|-------------|-------------|-----|--------|
| C1 | Agent XCM Protocol extension RFC | 855 | Spec phase |
| C2 | DeFi Agent Space runtime (reference implementation) | 1,430 | Dev phase |
| C3 | Governance Agent Space runtime | 1,430 | Dev phase |
| C4 | Creative Agent Space runtime | 1,145 | Dev phase |
| C5 | Cross-space agent communication demo | 570 | Dev phase |
| C6 | Security audit (Agent Registry + Agent Spaces) | 715 | Audit phase |
| C7 | JAM compatibility assessment + migration path | 285 | Research |

## Evaluation Criteria

Each child bounty is evaluated against:

1. **Code quality** — Clean, tested, documented Rust/TypeScript code
2. **Specification adherence** — Matches approved RFCs and standards
3. **Test coverage** — Unit tests + integration tests (Chopsticks/Zombienet)
4. **Documentation** — Developer guide included with every deliverable
5. **Community review** — Public review period before curator approval

## Open Participation

Child bounties in Tracks A, B, and C are **open to any team or individual** in the Polkadot ecosystem. You don't need to be the original proposer to claim a child bounty. This is designed to be a community effort.

**How to claim a child bounty:**
1. Review the deliverable specification
2. Post your intent and qualifications on the forum thread
3. Get curator approval
4. Deliver and submit for review

## Reporting

- **Monthly updates** posted to the Polkadot Forum
- **All code** open source on GitHub from day one
- **Financial transparency** — all payments tracked in public spreadsheet
- **Milestone demos** — live demonstrations for each completed child bounty

## Timeline

| Month | Milestones |
|-------|-----------|
| 1 | A1, A2 deployed. B1 spec draft published for review. |
| 2 | A3 first update cycle. B2 development begins. C1 RFC draft. |
| 3 | B1 finalized. B2 first testnet deployment. C2 development begins. |
| 4 | A4 parachain extensions (first batch). C3, C4 development begins. |
| 5 | B3 client library. C2, C3 testnet deployment. C5 demo. |
| 6 | C6 security audit begins. C7 JAM assessment. Final reporting. |

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| AI agents generating correct Polkadot code | 80%+ accuracy | Community survey + automated testing |
| SKILL.md adoption by AI tools | 3+ major tools | Directory listings + tool documentation |
| Agent Registry registrations (testnet) | 100+ agents | On-chain query |
| Agent Spaces deployed (testnet) | 3 reference spaces | Testnet verification |
| Coretime purchased for Agent Spaces | 3+ cores | Broker pallet queries |
| Community contributors | 10+ individuals | GitHub contributor count |
| GitHub stars | 200+ | Repository metrics |

## Why a Bounty (Not Direct Treasury Spend)

A bounty structure is superior for this project because:

1. **Multiple deliverables** — 18 child bounties across 3 tracks. Individual referenda for each would be governance-prohibitive.
2. **Open participation** — Any developer can claim child bounties. This isn't a single-team monopoly.
3. **Phased funding** — Curator releases funds per milestone, not upfront. Lower risk for the treasury.
4. **Flexibility** — If priorities shift (e.g., JAM launches early), the curator can redirect remaining child bounties.
5. **Accountability** — Curator deposit + public reporting creates clear accountability.

## Prior Art (Successful Polkadot Bounties)

- **Open Source Developer Grants (Bounty #59)** — $218,000 granted, 9 projects accepted. Similar model of curator-managed child bounties for developer tooling.
- **Anti-Scam Bounty** — Community-managed bounty with transparent reporting. Demonstrated the bounty model works for ongoing ecosystem maintenance.

---

## Appendix: PRC-8004 Draft Outline

The Polkadot Request for Comments (PRC) 8004 will specify:

### Identity Registry
- Agent DID (Decentralized Identifier) anchored to Polkadot account
- Metadata: name, description, capabilities, supported XCM versions
- Optional Proof-of-Personhood attestation (human-controlled vs autonomous)
- XCM-queryable from any parachain

### Reputation Registry
- Task completion scores (per-space, aggregated cross-space)
- Slashing history
- Uptime/availability metrics
- Peer endorsements (signed attestations from other agents)

### Validation Registry
- Cryptographic proofs of computation (task outputs)
- Dispute resolution protocol (challenger-defender with bond)
- Arbitration by space governance (OpenGov track per space)

### Agent XCM Extensions
- `AgentTaskRequest` — delegate a task to an agent on another chain
- `AgentTaskResponse` — return task result with validation proof
- `AgentReputationQuery` — query agent's cross-chain reputation score
- `AgentPayment` — escrow-based payment released on task validation
