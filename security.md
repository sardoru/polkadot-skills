# Polkadot Security Checklist

## Substrate / Pallet Security

### Storage

- [ ] **All collections are bounded** — Use `BoundedVec`, `BoundedBTreeMap`, never unbounded `Vec` or `BTreeMap` in storage
- [ ] **Correct key hashing** — `Blake2_128Concat` for user-controlled keys (prevents hash collision attacks), `Twox64Concat` only for system-controlled keys
- [ ] **No storage bloat vectors** — Every storage write must be bounded; deposits required for user-created entries
- [ ] **Storage deposits** — Charge deposits for storage created by user actions (similar to existential deposit)

### Extrinsics

- [ ] **Benchmarked weights** — Every dispatchable has weights from `#[benchmarks]`, never hardcoded
- [ ] **No unbounded computation** — Loops must have known upper bounds; use pagination for large iterations
- [ ] **Origin checks** — Every extrinsic verifies origin (`ensure_signed`, `ensure_root`, custom origins)
- [ ] **No panics** — No `unwrap()`, `expect()`, or `panic!()` in dispatchable calls; use `ensure!()` and `?`
- [ ] **Saturating/checked arithmetic** — Use `.saturating_add()`, `.checked_mul()`, never raw `+` / `*` on balances
- [ ] **Transactional safety** — State changes revert on error (FRAME does this by default; verify for manual storage writes)

### Economic Security

- [ ] **Deposit requirements** — Permissionless actions (creating accounts, storing data) require deposits
- [ ] **Fee alignment** — Extrinsic fees proportional to computational cost; no underpriced heavy operations
- [ ] **Slashing conditions** — Bad actors lose stake; slashing conditions are clear and tested
- [ ] **No free state mutation** — Every state change costs the caller (weight fees, deposits, or both)

### Migrations

- [ ] **try-runtime tested** — All migrations tested against live state with `pre_upgrade` / `post_upgrade`
- [ ] **Versioned** — Migrations check and update `StorageVersion`; idempotent if run twice
- [ ] **Weight budget** — Migration weight fits within a single block; use multi-block migrations for large state changes
- [ ] **Rollback plan** — Know what happens if migration fails mid-way

## ink! Contract Security

### Reentrancy

- [ ] **Checks-effects-interactions** — Update state before making external calls
- [ ] **No cross-contract callbacks that modify state** — Or use reentrancy guards

```rust
// BAD: state change after external call
#[ink(message)]
pub fn withdraw(&mut self, amount: Balance) {
    self.transfer(caller, amount); // external call
    self.balances.insert(caller, &0); // state change AFTER call
}

// GOOD: state change before external call
#[ink(message)]
pub fn withdraw(&mut self, amount: Balance) {
    let balance = self.balances.get(caller).unwrap_or(0);
    ensure!(balance >= amount, Error::InsufficientBalance);
    self.balances.insert(caller, &(balance - amount)); // state change FIRST
    self.transfer(caller, amount); // then external call
}
```

### Input Validation

- [ ] **Validate all inputs** — Don't trust caller-provided data
- [ ] **Zero-address checks** — Reject AccountId(0) where inappropriate
- [ ] **Bounds checking** — Validate array indices, amounts, percentages
- [ ] **Integer overflow** — Use `checked_*` methods for arithmetic on user inputs

### Access Control

- [ ] **Owner/admin patterns** — Critical functions gated by ownership checks
- [ ] **No hardcoded accounts** — Admin addresses configurable, not compiled in
- [ ] **Upgradability considered** — If contract is upgradeable, upgrade path is secure (proxy pattern)

## XCM Security

- [ ] **Barrier configured** — Only allow expected message types from expected origins
- [ ] **Weight limits set** — `Transact` calls specify realistic `require_weight_at_most`
- [ ] **Asset filters explicit** — Don't use `Wild(All)` in production barriers; specify exact assets
- [ ] **Reserve/teleport trust** — Only trust chains you've verified as reserves; misconfigured trust = loss of funds
- [ ] **Error handlers** — Include `SetErrorHandler` and `ReportError` for diagnosing failures
- [ ] **Fee payment** — Ensure `PayFees` (v5) or `BuyExecution` (legacy) uses adequate fees; underpayment = dropped message
- [ ] **Empowered Origins (v5)** — Review barriers for new XCM v5 origin types that can execute privileged operations cross-chain
- [ ] **Snowbridge V2** — If interacting with Ethereum via Snowbridge, validate bridge message origins and asset configurations

```rust
// BAD: Overly permissive barrier
pub type Barrier = AllowUnpaidExecutionFrom<Everything>;

// GOOD: Specific barriers
pub type Barrier = (
    TakeWeightCredit,
    AllowTopLevelPaidExecutionFrom<Everything>,
    AllowKnownQueryResponses<XcmPallet>,
    AllowSubscriptionsFrom<ParentOrSiblings>,
);
```

## Client-Side (PAPI / Frontend) Security

- [ ] **Validate chain metadata** — Ensure you're connected to the expected chain (check genesis hash)
- [ ] **Light client for trust** — Use Smoldot in production to avoid trusting centralized RPC nodes
- [ ] **Signer isolation** — Never expose private keys in frontend code; use extension signers
- [ ] **Transaction simulation** — Dry-run transactions before signing to catch errors early
- [ ] **No hardcoded indices** — Use typed PAPI descriptors, never hardcode pallet/call indices

## Operational Security

- [ ] **No sudo in production** — Use OpenGov for all privileged operations
- [ ] **Monitoring** — Monitor chain events for unexpected state changes
- [ ] **Upgrade process** — Runtime upgrades go through governance with adequate voting periods
- [ ] **Key management** — Validator and collator keys rotated regularly; session keys separate from stash
- [ ] **RPC exposure** — Production RPCs behind rate limiting; never expose unsafe RPC methods

## Common Vulnerability Patterns

| Vulnerability | Impact | Prevention |
|--------------|--------|------------|
| Unbounded iteration | DoS via weight exhaustion | Bound all loops, paginate |
| Missing origin check | Unauthorized access | Always check origin |
| Arithmetic overflow | Incorrect balances | Saturating/checked math |
| Storage bloat | Chain bloat, economic attack | Require deposits |
| XCM barrier bypass | Unauthorized cross-chain calls | Specific barrier config |
| Reentrancy (ink!) | Drained funds | Checks-effects-interactions |
| Migration failure | Bricked chain | try-runtime testing |
| Hardcoded keys | Single point of failure | Configurable via genesis/gov |
