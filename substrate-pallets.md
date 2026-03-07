# Substrate Pallet Development

## When to Use

Build a FRAME pallet when you need custom on-chain logic that runs as part of a parachain runtime. This is the most powerful and flexible option — you control storage, events, errors, calls, and hooks.

## Prerequisites

```bash
# Install Rust + Wasm target
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-unknown-unknown

# Install pop-cli for scaffolding
cargo install --locked pop-cli
```

## Scaffold a New Pallet

```bash
# Using pop-cli (recommended)
pop new pallet my-pallet

# Or manually in an existing runtime
mkdir pallets/my-pallet/src
```

## Pallet Structure (FRAME Macros)

The modern approach uses the unified `frame` crate. The legacy `#[frame_support::pallet]` still compiles but `#[frame::pallet]` is preferred for new code.

```rust
#![cfg_attr(not(feature = "std"), no_std)]

pub use pallet::*;

#[frame::pallet]
pub mod pallet {
    use frame::prelude::*;

    #[pallet::pallet]
    pub struct Pallet<T>(_);

    /// Configuration trait — defines types this pallet depends on
    #[pallet::config]
    pub trait Config: frame_system::Config {
        /// The overarching event type
        type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;

        /// Maximum length for a name
        #[pallet::constant]
        type MaxNameLength: Get<u32>;
    }

    /// Storage: use typed storage items
    #[pallet::storage]
    pub type Something<T: Config> = StorageValue<_, u32>;

    #[pallet::storage]
    pub type SomethingMap<T: Config> = StorageMap<
        _,
        Blake2_128Concat,  // Use Blake2_128Concat for user-controlled keys
        T::AccountId,
        BoundedVec<u8, T::MaxNameLength>,
    >;

    /// Events: always include relevant data for indexers
    #[pallet::event]
    #[pallet::generate_deposit(pub(super) fn deposit_event)]
    pub enum Event<T: Config> {
        /// Something was stored. [value, who]
        SomethingStored { value: u32, who: T::AccountId },
    }

    /// Errors: be specific
    #[pallet::error]
    pub enum Error<T> {
        /// Value was None when it should exist
        NoneValue,
        /// Overflow when incrementing
        StorageOverflow,
    }

    /// Dispatchable calls
    #[pallet::call]
    impl<T: Config> Pallet<T> {
        /// Store a value. Must have benchmarked weight.
        #[pallet::call_index(0)]
        #[pallet::weight(T::WeightInfo::do_something())]
        pub fn do_something(origin: OriginFor<T>, value: u32) -> DispatchResult {
            let who = ensure_signed(origin)?;
            Something::<T>::put(value);
            Self::deposit_event(Event::SomethingStored { value, who });
            Ok(())
        }
    }
}
```

## Storage Types

| Type | Use Case | Key Hashing |
|------|----------|-------------|
| `StorageValue` | Single values | N/A |
| `StorageMap` | Key-value mapping | `Blake2_128Concat` for user keys, `Twox64Concat` for system keys |
| `StorageDoubleMap` | Two-key mapping | Same hashing rules |
| `StorageNMap` | N-key mapping | Same hashing rules |
| `CountedStorageMap` | Map with count tracking | Same hashing rules |

**Important:** Always use `BoundedVec`, `BoundedBTreeMap`, etc. — never unbounded collections in storage. This prevents DoS attacks via storage bloat.

## Benchmarking (Required)

Every extrinsic MUST have benchmarked weights. Never use hardcoded weights in production.

```rust
#[benchmarks]
mod benchmarks {
    use super::*;

    #[benchmark]
    fn do_something() {
        let value = 100u32;
        let caller: T::AccountId = whitelisted_caller();

        #[extrinsic_call]
        _(RawOrigin::Signed(caller), value);

        assert_eq!(Something::<T>::get(), Some(value));
    }
}
```

Run benchmarks:
```bash
# Generate weights
./target/release/my-node benchmark pallet \
    --chain dev \
    --pallet pallet_my_pallet \
    --extrinsic "*" \
    --output pallets/my-pallet/src/weights.rs
```

## Storage Migrations

Any change to storage layout requires a migration. Skipping this will brick your chain.

The preferred modern pattern uses `VersionedMigration` + `UncheckedOnRuntimeUpgrade`, which handles version checking automatically:

```rust
pub mod v1 {
    use super::*;

    /// Inner migration logic — implement UncheckedOnRuntimeUpgrade (not OnRuntimeUpgrade)
    pub struct InnerMigrateToV1<T>(PhantomData<T>);

    impl<T: Config> UncheckedOnRuntimeUpgrade for InnerMigrateToV1<T> {
        fn on_runtime_upgrade() -> Weight {
            let count = OldStorage::<T>::drain().fold(0u64, |acc, (key, old_value)| {
                let new_value = transform(old_value);
                NewStorage::<T>::insert(key, new_value);
                acc + 1
            });
            T::DbWeight::get().reads_writes(count, count + 1)
        }

        #[cfg(feature = "try-runtime")]
        fn pre_upgrade() -> Result<Vec<u8>, DispatchError> {
            let count = OldStorage::<T>::iter().count() as u32;
            Ok(count.encode())
        }

        #[cfg(feature = "try-runtime")]
        fn post_upgrade(state: Vec<u8>) -> Result<(), DispatchError> {
            let old_count: u32 = Decode::decode(&mut &state[..]).expect("valid state");
            let new_count = NewStorage::<T>::iter().count() as u32;
            assert_eq!(old_count, new_count);
            Ok(())
        }
    }

    /// Wrap with VersionedMigration — handles version check + update automatically
    pub type MigrateToV1<T> = VersionedMigration<
        0,                    // From version
        1,                    // To version
        InnerMigrateToV1<T>,  // Inner migration
        Pallet<T>,            // Pallet
        <T as frame_system::Config>::DbWeight,
    >;
}
```

For large migrations that don't fit in a single block, use **multi-block migrations** via `pallet-migrations` and the `SteppedMigration` trait.

## Hooks

```rust
#[pallet::hooks]
impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
    fn on_initialize(_n: BlockNumberFor<T>) -> Weight {
        // Runs at start of every block. Return consumed weight.
        Weight::zero()
    }

    fn on_finalize(_n: BlockNumberFor<T>) {
        // Runs at end of every block. Cannot return errors.
    }

    fn offchain_worker(_n: BlockNumberFor<T>) {
        // Runs after block import. Has access to offchain storage and HTTP.
    }
}
```

## Testing

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use frame_support::{assert_ok, assert_noop};
    use sp_io::TestExternalities;

    // Build test externalities with mock runtime
    fn new_test_ext() -> TestExternalities {
        let t = frame_system::GenesisConfig::<Test>::default()
            .build_storage()
            .unwrap();
        TestExternalities::new(t)
    }

    #[test]
    fn it_works() {
        new_test_ext().execute_with(|| {
            assert_ok!(MyPallet::do_something(RuntimeOrigin::signed(1), 42));
            assert_eq!(Something::<Test>::get(), Some(42));
        });
    }

    #[test]
    fn errors_correctly() {
        new_test_ext().execute_with(|| {
            assert_noop!(
                MyPallet::fail_something(RuntimeOrigin::signed(1)),
                Error::<Test>::NoneValue
            );
        });
    }
}
```

## Checklist Before Shipping

- [ ] All storage items use bounded types
- [ ] All extrinsics have benchmarked weights
- [ ] Storage migrations written and tested with `try-runtime`
- [ ] Events emitted for all state changes
- [ ] Errors are specific and documented
- [ ] Call indices are explicit (`#[pallet::call_index(N)]`)
- [ ] Key hashing is appropriate (Blake2_128Concat for user-controlled keys)
- [ ] No `panic!` or `unwrap()` in dispatchable calls — use `ensure!` and `?`
- [ ] Use `#[frame::pallet(dev_mode)]` during prototyping (relaxes weight requirements), remove before production
