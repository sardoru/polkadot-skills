# Agile Coretime

## What It Is

Agile Coretime replaced Polkadot's slot auctions. Blockspace is now a commodity you purchase — on-demand for burst usage, or in bulk for sustained throughput. The Broker pallet on the Coretime chain manages allocation.

**Never reference "slot auctions" or "crowdloans" for new projects.** That system is deprecated.

## Two Modes

### On-Demand Coretime

- Pay per block when you need it
- Best for: low-traffic chains, testing, MVPs, intermittent workloads
- No upfront commitment
- Blocks are produced only when transactions exist

### Bulk Coretime

- Purchase a "core" for a fixed period (currently 28 days)
- Best for: production parachains with consistent throughput needs
- Can be split, shared, or resold on secondary markets
- Purchased via the Broker pallet on the Coretime chain

## Using On-Demand Coretime

On-demand coretime requires explicit order placement. A parachain (or an account acting on its behalf) must submit an order extrinsic per block:

- `OnDemandAssignmentProvider.placeOrderAllowDeath` — order, allow account to be reaped
- `OnDemandAssignmentProvider.placeOrderKeepAlive` — order, keep account alive

Blocks are produced only after an order is placed and a core is assigned.

```typescript
// Check on-demand pricing via PAPI
const api = client.getTypedApi(coretimeDescriptors)
const price = await api.query.OnDemandAssignmentProvider.SpotPrice.getValue()
```

## Purchasing Bulk Coretime

### Via PAPI

```typescript
import { coretime } from "@polkadot-api/descriptors"

const api = client.getTypedApi(coretime)

// Purchase a core during a sale
const tx = api.tx.Broker.purchase({
    price_limit: 10_000_000_000_000n, // Max price willing to pay
})
await tx.signAndSubmit(signer)
```

### Core Operations

```typescript
// Partition a core into two regions at a time pivot
const tx = api.tx.Broker.partition({
    region_id: regionId,
    pivot: 40, // Split at 40% of the region's time
})

// Assign a region to a parachain
const tx = api.tx.Broker.assign({
    region_id: regionId,
    task: 2000, // ParaId
    finality: "Final", // or "Provisional"
})

// Transfer a region to another account
const tx = api.tx.Broker.transfer({
    region_id: regionId,
    new_owner: recipientAddress,
})

// Place a region on the secondary market (interlace)
const tx = api.tx.Broker.interlace({
    region_id: regionId,
    pivot: CoreMask, // Which parts of the core to split
})
```

## Sale Lifecycle

1. **Interlude** — Renewals only (existing core holders get priority)
2. **Leadin** — Price starts high, decreases linearly (Dutch auction)
3. **Fixed Price** — After leadin, price stabilizes at the floor
4. **Sale Ends** — Unsold cores go to on-demand pool

```typescript
// Query current sale status
const saleInfo = await api.query.Broker.SaleInfo.getValue()
// {
//   saleStart: blockNumber,
//   leadinLength: blocks,
//   endPrice: bigint,
//   regionBegin: timeslice,
//   regionEnd: timeslice,
//   coresOffered: number,
//   coresSold: number,
// }
```

## For JAM Preparation

JAM (Join-Accumulate Machine) will extend the coretime model. Design your application with these principles:

- **Stateless execution** — JAM services should not depend on persistent on-chain state during execution
- **Work packages** — Think of your computation as discrete work packages that can be verified
- **Core-agnostic** — Don't assume you'll always have the same core; design for dynamic allocation

## Key Parameters

| Parameter | Current Value | Note |
|-----------|--------------|------|
| Bulk period | 28 days | Region duration |
| Timeslice | 80 blocks (~8 min) | Smallest unit of coretime |
| Cores available | ~18-25 | Varies per sale |
| On-demand price | Dynamic | Set by demand |

## Common Patterns

### MVP Launch (On-Demand)
1. Register parachain with Polkadot
2. Run collator with on-demand config
3. Pay per block — ideal for testing product-market fit

### Production Launch (Bulk)
1. Purchase bulk core during sale
2. Assign to your ParaId
3. Renew each period during interlude phase (priority pricing)

### Elastic Scaling
1. Hold one bulk core for baseline
2. Use on-demand for burst traffic
3. Purchase additional cores during high-demand periods
