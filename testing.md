# Testing on Polkadot

## Testing Hierarchy

| Level | Tool | Speed | Fidelity |
|-------|------|-------|----------|
| Unit tests | `#[test]` / `#[ink::test]` | Fast | Low |
| Fork tests | Chopsticks | Medium | High |
| Integration tests | Zombienet | Slow | Highest |
| Migration tests | try-runtime | Medium | High |

Always test at the lowest level that catches the bug. Escalate to higher fidelity only when needed.

## Chopsticks — Fork-Based Testing

Chopsticks forks a live chain locally. Use it for:
- Testing extrinsics against real state
- XCM testing between chains
- Runtime upgrade dry-runs
- Reproducing production bugs

### Setup

```bash
npm install -g @acala-network/chopsticks
```

### Single Chain Fork

```bash
# Fork Polkadot at latest block
chopsticks --config=polkadot

# Fork at specific block
chopsticks --config=polkadot --block 20000000

# Fork with custom runtime (for testing upgrades)
chopsticks --config=polkadot --wasm-override ./target/release/wbuild/polkadot-runtime/polkadot_runtime.wasm
```

### Multi-Chain XCM Testing

```bash
# Fork Polkadot + Asset Hub for XCM testing
chopsticks xcm -r polkadot -p polkadot-asset-hub
```

### Config File

```yaml
# chopsticks.yml
endpoint: wss://rpc.polkadot.io
port: 8000
block: null # latest
db: ./db.sqlite

# Override storage for testing
import-storage:
  System:
    Account:
      - - - "5GrwvaEF..."
        - data:
            free: "100000000000000000"
  Sudo:
    Key: "5GrwvaEF..."
```

### Chopsticks with TypeScript

```typescript
import { setupContext } from "@acala-network/chopsticks-testing"

const context = await setupContext({
  endpoint: "wss://rpc.polkadot.io",
  blockNumber: 20000000,
})

// Impersonate any account
await context.dev.setStorage({
  System: {
    Account: [
      [["5GrwvaEF..."], { data: { free: "1000000000000000" } }],
    ],
  },
})

// Create new block
await context.dev.newBlock()
```

## Zombienet — Multi-Node Integration

Zombienet spawns real validator and collator networks. Use it for:
- Full consensus testing
- Parachain onboarding flows
- XCM end-to-end with real message passing
- Performance/stress testing

### Setup

```bash
# Install
npm install -g @zombienet/cli

# Or download binary
# https://github.com/paritytech/zombienet/releases
```

### Network Config

```toml
# zombienet.toml
[settings]
timeout = 600

[relaychain]
chain = "paseo-local"
default_command = "./target/release/polkadot"
default_args = ["-lparachain=debug"]

[[relaychain.nodes]]
name = "alice"
validator = true

[[relaychain.nodes]]
name = "bob"
validator = true

[[parachains]]
id = 1000
chain = "asset-hub-paseo-local"

[[parachains.collators]]
name = "asset-hub-collator"
command = "./target/release/polkadot-parachain"

[[parachains]]
id = 2000
chain = "my-parachain-local"

[[parachains.collators]]
name = "my-collator"
command = "./target/release/my-parachain-node"
```

### Run

```bash
zombienet spawn zombienet.toml --provider native

# With tests
zombienet test zombienet.toml zombienet-tests.zndsl
```

### Test DSL

```
# zombienet-tests.zndsl
Description: Basic XCM test
Network: ./zombienet.toml
Creds: config

alice: is up
bob: is up
asset-hub-collator: is up
alice: parachain 1000 is registered within 120 seconds
alice: parachain 2000 is registered within 120 seconds
alice: parachain 1000 block height is at least 5 within 200 seconds
```

## try-runtime — Migration Testing

Test runtime upgrades against real chain state without risking production.

```bash
# Test migration against live chain
cargo build --release --features try-runtime

./target/release/my-node try-runtime \
    --runtime ./target/release/wbuild/my-runtime/my_runtime.wasm \
    on-runtime-upgrade \
    live --uri wss://rpc.polkadot.io
```

This executes:
1. `pre_upgrade()` hooks
2. The actual migration
3. `post_upgrade()` hooks
4. Reports weight usage and storage changes

## Unit Testing Patterns

### Pallet Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use frame_support::{assert_ok, assert_noop, parameter_types};
    use sp_io::TestExternalities;
    use sp_runtime::BuildStorage;

    // Mock runtime
    frame_support::construct_runtime!(
        pub enum Test {
            System: frame_system,
            Balances: pallet_balances,
            MyPallet: pallet_my_pallet,
        }
    );

    // Configure mock runtime
    parameter_types! {
        pub const BlockHashCount: u64 = 250;
        pub const ExistentialDeposit: u128 = 1;
    }

    impl frame_system::Config for Test {
        type Block = frame_system::mocking::MockBlock<Test>;
        // ... minimal config
    }

    impl pallet_balances::Config for Test {
        type Balance = u128;
        type ExistentialDeposit = ExistentialDeposit;
        // ... minimal config
    }

    impl pallet_my_pallet::Config for Test {
        type RuntimeEvent = RuntimeEvent;
        // ... your config
    }

    fn new_test_ext() -> TestExternalities {
        let mut t = frame_system::GenesisConfig::<Test>::default()
            .build_storage()
            .unwrap();

        pallet_balances::GenesisConfig::<Test> {
            balances: vec![(1, 1000), (2, 2000)],
        }
        .assimilate_storage(&mut t)
        .unwrap();

        TestExternalities::new(t)
    }

    #[test]
    fn test_something() {
        new_test_ext().execute_with(|| {
            System::set_block_number(1);
            assert_ok!(MyPallet::do_something(RuntimeOrigin::signed(1), 42));
            System::assert_last_event(Event::SomethingStored { value: 42, who: 1 }.into());
        });
    }
}
```

### ink! Contract Tests

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[ink::test]
    fn constructor_works() {
        let contract = MyContract::new(42);
        assert_eq!(contract.get_value(), 42);
    }

    #[ink::test]
    fn set_value_works() {
        let mut contract = MyContract::default();
        assert!(contract.set_value(42).is_ok());
        assert_eq!(contract.get_value(), 42);
    }
}
```

## Best Practices

1. **Write unit tests first** — They're fast and catch most logic bugs
2. **Use Chopsticks for XCM** — Fork both chains and test the full message flow
3. **Always try-runtime before upgrading** — One migration bug can brick your chain
4. **Zombienet for CI** — Run integration tests in CI before merging parachain changes
5. **Test with real state** — Chopsticks lets you fork mainnet; use it to catch state-dependent bugs
6. **Check events in tests** — Don't just check storage; verify correct events are emitted
