# Polkadot Agent Mesh — Proposal

## What Ethereum Is Doing (and Where They're Weak)

The EF strategy has 3 pillars:

1. **ERC-8004** — on-chain registries for agent identity (NFT-based handles), reputation (feedback signals), and validation (cryptographic proofs)
2. **Coordination layer** — agents discover each other, route payments, verify outcomes. Heavy compute stays off-chain
3. **"Props AI"** — privacy, censorship resistance, security for agents (device-local processing, key-based auth over human-based auth)

**Their gap:** Ethereum is a single execution environment. Agents all share one congested chain (or fragment across L2s with no native coordination). There's no concept of isolated execution domains with custom rules — every agent plays by the same global rules.

## Why Polkadot Is Architecturally Superior for This

Polkadot already has what Ethereum is trying to retrofit:

| Capability                    | Ethereum             | Polkadot                |
| ----------------------------- | -------------------- | ----------------------- |
| Isolated execution domains    | No (L2 fragmentation) | Yes — Parachains / Coretime |
| Custom rule sets per domain   | No (one EVM)          | Yes — Per-parachain runtime |
| Native cross-domain messaging | No (bridges)          | Yes — XCM                   |
| On-chain governance           | No (off-chain)        | Yes — OpenGov               |
| Flexible compute allocation   | No                    | Yes — Agile Coretime        |
| Proof-of-Personhood           | No                    | Yes — Gavin's active bet    |

**The pitch:** Ethereum can be a trust layer. Polkadot can be a **trust ecosystem** — where each community, DAO, or industry vertical gets its own agent execution environment with custom rules, native interop, and shared security.

## The Product: "Agent Spaces"

**Concept:** Dedicated blockspaces (parachains or coretime allocations) purpose-built for agent-to-agent economies, each with governance-defined rules.

### Three Tiers

#### 1. Agent Registry Chain (system parachain)

- Polkadot-native agent identity standard (**PRC-8004**, inspired by ERC-8004)
- Identity registry, reputation registry, validation registry
- XCM-accessible from any parachain
- Proof-of-Personhood integration (Gavin's work) to distinguish human-controlled vs autonomous agents

#### 2. Agent Spaces (coretime-powered)

- Any DAO, community, or enterprise can spin up an "Agent Space" via coretime
- Custom rules: who can register agents, fee structures, dispute resolution, allowed operations
- Examples:
  - **DeFi Agent Space** — agents that trade, lend, rebalance. Rules: max position sizes, circuit breakers, slashing for bad actors
  - **Governance Agent Space** — delegate voting, proposal analysis. Rules: identity-verified only, human-override mandatory
  - **Supply Chain Agent Space** — procurement, logistics. Rules: KYB-verified companies only, SLA enforcement
  - **Creative Agent Space** — content creation, licensing. Rules: attribution enforcement, royalty distribution

#### 3. Agent XCM Protocol (cross-space communication)

- Extend XCM for agent-to-agent messages (task delegation, payment routing, reputation queries)
- Agents in one space can hire agents in another space, with escrow and verification
- This is the "Google Reviews + payment rails" Crapis described, but natively cross-chain

## Why Gavin / Parity Would Back This

- **JAM alignment** — Agent Spaces are a killer use case for JAM's "decentralized supercomputer" vision
- **Coretime revenue** — every Agent Space = coretime demand
- **Proof-of-Personhood synergy** — Gavin's PoP research gets a concrete product use case (human vs agent distinction)
- **Narrative win** — positions Polkadot as the "agent-native" chain vs Ethereum's "agent-compatible" chain

## Competitive Framing

> "Ethereum gives agents a shared apartment. Polkadot gives them a city — with neighborhoods, local laws, and a transit system."

## Next Steps

- [ ] Define PRC-8004 spec (Polkadot-native agent identity standard)
- [ ] Design Agent XCM message format extensions
- [ ] Prototype Agent Registry Chain runtime (Substrate)
- [ ] Build reference Agent Space (DeFi or Governance)
- [ ] Draft governance proposal for system parachain status
- [ ] Outreach to Parity / W3F for alignment on JAM integration
