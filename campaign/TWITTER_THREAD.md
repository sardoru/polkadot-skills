# Twitter/X Thread — Polkadot Agent Mesh Campaign

Post from your account (or pitch to @Polkadot). Each numbered block = one tweet.

---

**1/12**

Solana just made their entire dev stack agent-native with SKILL.md.

Any AI coding agent that reads it builds on Solana correctly. Zero hallucination. Zero outdated APIs.

Polkadot has nothing like this. Until now.

Introducing: Polkadot Agent Mesh

Thread ->

---

**2/12**

The problem: When an AI agent tries to build on Polkadot today, it:

- Uses @polkadot/api (maintenance mode) instead of PAPI
- References slot auctions (deprecated)
- Writes XCM v3 (2 versions behind)
- Hallucinates npm packages that don't exist

Every broken build = a developer lost to Solana.

---

**3/12**

We built the fix: a complete, validated SKILL.md for Polkadot.

12 documents. 2,270 lines. Every API cross-referenced against live sources.

One file that tells any AI agent exactly how to build on Polkadot.

---

**4/12**

Opinionated stack decisions (non-negotiable):

- PAPI first (not polkadot.js)
- XCM v5 (PayFees, InitiateTransfer)
- ink! + pop-cli for contracts
- Chopsticks + Zombienet for testing
- Agile Coretime (not auctions)
- #[frame::pallet] (unified frame crate)

---

**5/12**

Validation was brutal:

6 parallel research agents checked every claim against:
- papi.how
- use.ink
- Polkadot SDK repo
- wiki.polkadot.network
- npm registry

44 corrections applied. Zero hallucinated package names remain.

---

**6/12**

But the SKILL.md is just Phase 1.

Phase 2 is where it gets interesting: Agent Spaces.

Ethereum just announced ERC-8004 — their strategy to become the "trust layer for AI agents."

Here's the problem: Ethereum is a single execution environment.

---

**7/12**

Polkadot already has what Ethereum is trying to retrofit:

- Isolated execution domains (parachains)
- Custom rules per domain (per-chain runtime)
- Native cross-chain messaging (XCM)
- Flexible compute allocation (Agile Coretime)
- On-chain governance (OpenGov)
- Proof-of-Personhood (Gavin's active research)

---

**8/12**

Agent Spaces: dedicated blockspaces for agent-to-agent economies.

Tier 1: Agent Registry Chain (system parachain)
- PRC-8004: agent identity, reputation, validation
- Proof-of-Personhood: human vs autonomous distinction

Tier 2: Agent Spaces (coretime-powered)
- Custom rules per vertical
- Any DAO can spin one up

---

**9/12**

Example Agent Spaces:

DeFi: agents that trade, lend, rebalance. Circuit breakers + slashing.

Governance: delegate voting, proposal analysis. Identity-verified only.

Supply Chain: procurement, logistics. KYB-verified companies only.

Creative: content licensing, royalty distribution.

---

**10/12**

Tier 3: Agent XCM Protocol

Agents in one Space can hire agents in another Space.

Task delegation. Payment routing. Reputation queries. Escrow + verification.

Cross-chain agent coordination — natively. No bridges. No fragmentation.

---

**11/12**

The pitch:

"Ethereum gives agents a shared apartment.

Polkadot gives them a city — with neighborhoods, local laws, and a transit system."

Every Agent Space = coretime demand.
Every coretime purchase = treasury funding.
It's a flywheel.

---

**12/12**

We're submitting a Polkadot Treasury proposal.

Phase 1: Ship SKILL.md ($10K)
Phase 2: Agent Spaces spec + reference impl ($40K)
Phase 3: Production launch ($100K)

Want to help?

- Review the docs (open source)
- Comment on the forum post [link]
- Vote Aye when it goes live

#PolkadotAgentMesh #Polkadot #JAM

---

## Engagement Notes

- Pin the thread to your profile
- Quote-tweet the Solana SKILL.md announcement for context
- Tag @paboracle @nicepicks @nicepicks_w3f
- Post during US morning / EU afternoon overlap (14:00-16:00 UTC)
- Respond to every reply within the first hour for algorithm boost
