# XCM — Cross-Consensus Messaging

## What XCM Is

XCM (Cross-Consensus Message Format) is Polkadot's universal language for cross-chain communication. It lets parachains, relay chains, and external consensus systems exchange messages — transfers, remote calls, governance actions.

**Use XCM v5.** Earlier versions (v4 and below) are deprecated. XCM v5 shipped with Polkadot 2.0 in mid-2025.

## Core Concepts

### Locations (renamed from MultiLocation to Location in v4+)

Everything in XCM is addressed by location, relative to the sender:

```
// Relay chain (from a parachain's perspective)
Location { parents: 1, interior: Here }

// Parachain 1000 (Asset Hub, from relay chain)
Location { parents: 0, interior: X1(Parachain(1000)) }

// An account on Asset Hub (from relay chain)
Location {
    parents: 0,
    interior: X2(Parachain(1000), AccountId32 { id: [..], network: None })
}

// Native DOT (from any parachain)
Location { parents: 1, interior: Here }

// USDT on Asset Hub (from any parachain)
Location {
    parents: 1,
    interior: X3(Parachain(1000), PalletInstance(50), GeneralIndex(1984))
}
```

### Assets

```rust
// DOT from a parachain's view
Asset {
    id: AssetId(Location { parents: 1, interior: Here }),
    fun: Fungible(1_000_000_000_000), // 1 DOT
}

// Multiple assets
Assets::from(vec![
    (Location::parent(), 1_000_000_000_000u128).into(), // 1 DOT
])
```

### Instructions (Key Ones)

| Instruction | Purpose |
|-------------|---------|
| `WithdrawAsset` | Take assets from the origin account into holding |
| `DepositAsset` | Deposit assets to a beneficiary |
| `BuyExecution` | Pay for XCM execution weight (legacy, still works) |
| `PayFees` | **v5:** Improved fee payment — designate one asset for fees. Preferred over `BuyExecution` |
| `TransferAsset` | Withdraw + Deposit in one step |
| `InitiateTransfer` | **v5:** Unified cross-chain transfer (replaces `InitiateReserveWithdraw`/`InitiateTeleport`) |
| `Transact` | Execute an encoded call on the remote chain |
| `SetHints` | **v5:** Set metadata hints (e.g., `AssetClaimer` for trapped asset recovery) |
| `ExpectAsset` | Assert holding contains expected assets |
| `SetErrorHandler` | Define error recovery logic |
| `ReportError` | Send execution outcome to a destination |

## Common Patterns

### 1. Teleport Assets (Trusted Chains Only)

Used between relay chain <-> system parachains (Asset Hub, etc.)

```typescript
// From PAPI (TypeScript) — prefer transfer_assets (unified extrinsic)
const tx = api.tx.XcmPallet.transfer_assets({
    dest: XcmVersionedLocation.V5({
        parents: 0,
        interior: XcmV5Junctions.X1(XcmV5Junction.Parachain(1000)),
    }),
    beneficiary: XcmVersionedLocation.V5({
        parents: 0,
        interior: XcmV5Junctions.X1(
            XcmV5Junction.AccountId32({ network: undefined, id: recipientBytes })
        ),
    }),
    assets: XcmVersionedAssets.V5([{
        id: { parents: 0, interior: XcmV5Junctions.Here() },
        fun: XcmV5Fungibility.Fungible(amount),
    }]),
    fee_asset_item: 0,
    weight_limit: XcmV5WeightLimit.Unlimited(),
})
```

### 2. Reserve Transfer (Untrusted Chains)

Used between parachains that don't trust each other directly.

```rust
let message = Xcm(vec![
    WithdrawAsset(assets.clone()),
    InitiateReserveWithdraw {
        assets: Wild(All),
        reserve: dest.clone(),
        xcm: Xcm(vec![
            BuyExecution { fees: fee_asset, weight_limit: Unlimited },
            DepositAsset {
                assets: Wild(All),
                beneficiary: beneficiary.clone(),
            },
        ]),
    },
]);
```

### 3. Remote Transact (Execute Call on Another Chain)

```rust
let message = Xcm(vec![
    WithdrawAsset(fee_assets),
    BuyExecution { fees: fee_asset, weight_limit: Unlimited },
    Transact {
        origin_kind: OriginKind::SovereignAccount,
        require_weight_at_most: Weight::from_parts(1_000_000_000, 64 * 1024),
        call: encoded_call.into(),
    },
    ExpectTransactStatus(MaybeErrorCode::Success),
    RefundSurplus,
    DepositAsset {
        assets: Wild(All),
        beneficiary: sender_on_dest,
    },
]);
```

## XCM in Pallets (Sending from Runtime)

```rust
use xcm::latest::prelude::*; // resolves to v5
use pallet_xcm::ensure_response_from;

// Send XCM from your pallet
let dest: Location = Location::new(1, [Parachain(1000)]);
let message = Xcm(vec![
    // ... instructions
]);

pallet_xcm::Pallet::<T>::send_xcm(Here, dest, message)
    .map_err(|_| Error::<T>::XcmSendFailed)?;
```

## XCM Configuration (Runtime)

```rust
parameter_types! {
    pub const RelayNetwork: Option<NetworkId> = Some(NetworkId::Polkadot);
    pub RelayChainOrigin: RuntimeOrigin = cumulus_pallet_xcm::Origin::Relay.into();
    pub UniversalLocation: InteriorLocation = [GlobalConsensus(RelayNetwork::get().unwrap()), Parachain(ParachainInfo::parachain_id().into())].into();
}

pub struct XcmConfig;
impl xcm_executor::Config for XcmConfig {
    type RuntimeCall = RuntimeCall;
    type XcmSender = XcmRouter;
    type AssetTransactor = LocalAssetTransactor;
    type OriginConverter = XcmOriginToTransactDispatchOrigin;
    type IsReserve = NativeAsset; // Which assets this chain is reserve for
    type IsTeleporter = NativeAsset; // Which assets can be teleported
    type Barrier = Barrier; // Which messages are allowed
    type Weigher = FixedWeightBounds<UnitWeightCost, RuntimeCall, MaxInstructions>;
    type Trader = UsingComponents<WeightToFee, RelayLocation, AccountId, Balances, ()>;
    type TransactionalProcessor = FrameTransactionalProcessor; // Ensures atomic XCM execution
    // ... more config (Aliasers, CallDispatcher, SafeCallFilter, etc.)
}
```

## Testing XCM

**Always test XCM flows end-to-end.** XCM bugs in production are extremely costly.

### With Chopsticks (Fork-Based)

```bash
# Fork Polkadot + Asset Hub locally
npx @acala-network/chopsticks xcm \
    -r polkadot \
    -p polkadot-asset-hub
```

### With Zombienet (Multi-Node)

```toml
# zombienet.toml
[relaychain]
chain = "paseo-local"
default_command = "./target/release/polkadot"

[[relaychain.nodes]]
name = "alice"

[[relaychain.nodes]]
name = "bob"

[[parachains]]
id = 1000
chain = "asset-hub-paseo-local"

[[parachains.collators]]
name = "asset-hub"
command = "./target/release/polkadot-parachain"
```

```bash
zombienet spawn zombienet.toml --provider native
```

## Common Mistakes

1. **Not paying for execution** — Every XCM program on a remote chain must pay for execution via `PayFees` (v5) or `BuyExecution` (legacy)
2. **Wrong weight limits** — Underestimating `require_weight_at_most` in `Transact` causes silent failures
3. **Incorrect asset locations** — DOT is `Location::parent()` from a parachain, not `Here`
4. **Forgetting RefundSurplus** — Always include `RefundSurplus` + `DepositAsset` after `Transact` to recover unused fees
5. **Testing on wrong version** — Ensure your runtime's XCM config matches the version you're building messages for
