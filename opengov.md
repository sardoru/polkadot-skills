# OpenGov — On-Chain Governance

## What It Is

Polkadot OpenGov is fully on-chain governance. Every parameter change, treasury spend, runtime upgrade, and system decision goes through governance tracks with different approval thresholds. There is no council, no sudo — only token-weighted voting with conviction.

## Key Concepts

### Tracks

Each proposal type has a track with specific thresholds:

| Track | Use Case | Max Spend | Decision Period |
|-------|----------|-----------|----------------|
| Root | Runtime upgrades, critical changes | Unlimited | 28 days |
| Whitelisted Caller | Pre-approved technical calls | N/A | 28 days |
| Staking Admin | Staking parameters | N/A | 28 days |
| General Admin | General parameter changes | N/A | 28 days |
| Referendum Canceller | Cancel ongoing referenda | N/A | 7 days |
| Referendum Killer | Kill referenda, slash deposit | N/A | 28 days |
| Treasurer | Large treasury spends | 10,000,000 DOT | 28 days |
| Big Spender | Treasury spends | 10,000 DOT | 28 days |
| Medium Spender | Treasury spends | 1,000 DOT | 28 days |
| Small Spender | Treasury spends | 250 DOT | 28 days |
| Big Tipper | Tips | 1,000 DOT | 7 days |
| Small Tipper | Tips | 250 DOT | 7 days |

**Note:** OpenGov does NOT use static "supermajority" or "simple majority" thresholds. Each track has **dynamic approval and support curves** that change over the decision period. Approval starts at ~100% and decreases toward 50%; support thresholds decrease over time at track-specific rates.

### Conviction Voting

Voters can lock tokens for longer to multiply their voting power:

| Lock Period | Conviction Multiplier |
|-------------|----------------------|
| None (0)    | 0.1x                 |
| 1 period    | 1x                   |
| 2 periods   | 2x                   |
| 4 periods   | 3x                   |
| 8 periods   | 4x                   |
| 16 periods  | 5x                   |
| 32 periods  | 6x                   |

### Lifecycle

1. **Submission** — Proposal submitted with deposit
2. **Decision Deposit** — Someone places decision deposit to start voting
3. **Deciding** — Voting period begins
4. **Confirmation** — If passing threshold met, confirmation period starts
5. **Enacted** — Proposal executes after enactment period

## Submitting Proposals via PAPI

### Treasury Spend Proposal

```typescript
import { dot } from "@polkadot-api/descriptors"

const api = client.getTypedApi(dot)

// Preimage: the actual call to execute
const spendCall = api.tx.Treasury.spend_local({
    amount: 1_000_000_000_000_000n, // 1000 DOT
    beneficiary: MultiAddress.Id(beneficiaryAddress),
})

// Submit preimage first
const preimage = api.tx.Preimage.note_preimage({
    bytes: spendCall.encodedData,
})
await preimage.signAndSubmit(signer)

// Then submit referendum
const referendum = api.tx.Referenda.submit({
    proposal_origin: { system: "Origins", value: "MediumSpender" },
    proposal: {
        Lookup: {
            hash: preimageHash,
            len: preimageLen,
        },
    },
    enactment_moment: { After: 100 }, // Enact 100 blocks after approval
})
await referendum.signAndSubmit(signer)

// Place decision deposit
const deposit = api.tx.Referenda.place_decision_deposit({
    index: referendumIndex,
})
await deposit.signAndSubmit(signer)
```

### Voting

```typescript
// Vote Aye with 1x conviction
const vote = api.tx.ConvictionVoting.vote({
    poll_index: referendumIndex,
    vote: {
        Standard: {
            vote: { aye: true, conviction: "Locked1x" },
            balance: 100_000_000_000_000n, // 100 DOT
        },
    },
})
await vote.signAndSubmit(signer)

// Delegate voting power
const delegate = api.tx.ConvictionVoting.delegate({
    class: trackId, // Track to delegate for
    to: MultiAddress.Id(delegateAddress),
    conviction: "Locked1x",
    balance: 100_000_000_000_000n,
})
await delegate.signAndSubmit(signer)
```

### Query Referendum Status

```typescript
// Get referendum info
const info = await api.query.Referenda.ReferendumInfoFor.getValue(referendumIndex)

// Get all ongoing referenda
const ongoing = await api.query.Referenda.ReferendumInfoFor.getEntries()
const active = ongoing.filter(({ value }) => value.type === "Ongoing")

// Watch a specific referendum (Observable pattern)
api.query.Referenda.ReferendumInfoFor.watchValue(referendumIndex).subscribe((info) => {
    if (info.type === "Ongoing") {
        console.log("Ayes:", info.value.tally.ayes)
        console.log("Nays:", info.value.tally.nays)
    }
})
```

## Agent-Relevant Governance Patterns

### Governance Agent (Proposal Analyzer)

```typescript
// Fetch all active referenda and decode their preimages
async function analyzeActiveReferenda(api) {
    const referenda = await api.query.Referenda.ReferendumInfoFor.getEntries()

    for (const { keyArgs: [index], value: info } of referenda) {
        if (info.type !== "Ongoing") continue

        const { proposal, tally, deciding } = info.value

        // Decode the proposal preimage
        if (proposal.type === "Lookup") {
            const preimage = await api.query.Preimage.PreimageFor.getValue(
                [proposal.value.hash, proposal.value.len]
            )
            // Decode and analyze the call
        }

        console.log(`Referendum #${index}: Ayes=${tally.ayes}, Nays=${tally.nays}`)
    }
}
```

### Delegation Agent

```typescript
// Auto-delegate to trusted addresses per track
async function setupDelegation(api, signer, delegations: Record<number, string>) {
    const txs = Object.entries(delegations).map(([track, delegate]) =>
        api.tx.ConvictionVoting.delegate({
            class: Number(track),
            to: MultiAddress.Id(delegate),
            conviction: "Locked1x",
            balance: delegateBalance,
        })
    )

    // Batch all delegations
    const batch = api.tx.Utility.batch_all({ calls: txs.map(t => t.decodedCall) })
    await batch.signAndSubmit(signer)
}
```

## Best Practices

1. **Always use preimages** — Submit the call as a preimage first, reference by hash in the referendum
2. **Pick the right track** — Using a higher track than needed means stricter thresholds and longer periods
3. **Decision deposits** — Someone must place a decision deposit for voting to begin; include this in your flow
4. **Enactment delay** — Set appropriate delays; critical changes should have longer delays for safety
5. **Test on Westend** — Westend has OpenGov with low stakes; test governance flows there first
