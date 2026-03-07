# ink! Smart Contracts

## When to Use

Use ink! when you want to deploy smart contracts on Polkadot parachains that support the Contracts pallet (Astar, Aleph Zero, Phala, etc.). ink! compiles to Wasm and runs in a sandboxed environment.

**Status note (Jan 2026):** The original ink! team discontinued active development. The **ink! Alliance** has secured Polkadot treasury funding to stabilize ink! v6 for PolkaVM (RISC-V). The current stable release is **ink! 5.1.1**; ink! v6.0.0-beta targets the new pallet-revive + PolkaVM architecture. This guide covers the stable v5 path.

**Choose ink! over a pallet when:**
- You want permissionless deployment (no runtime upgrade needed)
- Your logic is application-specific, not chain-level
- You need rapid iteration without governance approval

## Prerequisites

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install pop-cli (recommended over cargo-contract)
cargo binstall pop-cli --locked
# Or: cargo install --locked pop-cli

# Or install cargo-contract directly
cargo install cargo-contract --locked
```

## Scaffold a New Contract

```bash
# Using pop-cli (recommended)
pop new contract my_contract

# Using cargo-contract
cargo contract new my_contract
```

## Contract Structure

```rust
#![cfg_attr(not(feature = "std"), no_std, no_main)]

#[ink::contract]
mod my_contract {
    use ink::storage::Mapping;

    /// Contract storage
    #[ink(storage)]
    pub struct MyContract {
        /// A simple value
        value: u32,
        /// A mapping from account to balance
        balances: Mapping<AccountId, Balance>,
    }

    /// Events
    #[ink(event)]
    pub struct ValueChanged {
        #[ink(topic)]  // indexed for filtering
        from: AccountId,
        value: u32,
    }

    /// Errors
    #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
    #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
    pub enum Error {
        /// Value overflow
        Overflow,
        /// Caller is not authorized
        Unauthorized,
        /// Insufficient balance
        InsufficientBalance,
    }

    pub type Result<T> = core::result::Result<T, Error>;

    impl MyContract {
        /// Constructor — called once on deployment
        #[ink(constructor)]
        pub fn new(init_value: u32) -> Self {
            Self {
                value: init_value,
                balances: Mapping::default(),
            }
        }

        /// Default constructor
        #[ink(constructor)]
        pub fn default() -> Self {
            Self::new(0)
        }

        /// Mutating message — changes state
        #[ink(message)]
        pub fn set_value(&mut self, new_value: u32) -> Result<()> {
            let caller = self.env().caller();
            self.value = new_value;
            self.env().emit_event(ValueChanged {
                from: caller,
                value: new_value,
            });
            Ok(())
        }

        /// Non-mutating message — reads state (free, no gas)
        #[ink(message)]
        pub fn get_value(&self) -> u32 {
            self.value
        }

        /// Payable message — accepts token transfers
        #[ink(message, payable)]
        pub fn deposit(&mut self) {
            let caller = self.env().caller();
            let amount = self.env().transferred_value();
            let current = self.balances.get(caller).unwrap_or(0);
            self.balances.insert(caller, &(current + amount));
        }
    }
}
```

## Storage Types

| Type | Use Case |
|------|----------|
| `ink::storage::Mapping<K, V>` | Key-value mapping (lazy-loaded) |
| `Vec<T>` | Small, bounded collections only |
| `ink::storage::Lazy<T>` | Large values loaded on demand |

**Rules:**
- Use `Mapping` for unbounded key-value data — it's lazy and efficient
- Never use `Vec` for unbounded data — it loads entirely into memory
- Use `Lazy` for large values that aren't always needed

## Cross-Contract Calls

```rust
use other_contract::OtherContractRef;

#[ink(storage)]
pub struct MyContract {
    other: OtherContractRef,
}

#[ink(message)]
pub fn call_other(&self) -> u32 {
    // Direct typed call
    self.other.get_value()
}

#[ink(message)]
pub fn call_other_with_gas(&self) -> u32 {
    // With explicit gas limit
    ink::env::call::build_call::<Environment>()
        .call(self.other_address)
        .exec_input(ink::env::call::ExecutionInput::new(
            ink::env::call::Selector::new(ink::selector_bytes!("get_value"))
        ))
        .returns::<u32>()
        .invoke()
}
```

## PSP Standards (ink! token standards)

| Standard | Purpose | Equivalent |
|----------|---------|------------|
| PSP22    | Fungible token | ERC-20 |
| PSP34    | Non-fungible token | ERC-721 |
| PSP37    | Multi-token | ERC-1155 |

Use the `pendzl` library for standard implementations (successor to the legacy `openbrush`):

```toml
# Cargo.toml
[dependencies]
pendzl = { version = "1.0", default-features = false, features = ["psp22"] }
```

## Build and Deploy

```bash
# Build (produces .wasm + metadata.json)
pop build

# Or with cargo-contract
cargo contract build --release

# Deploy to local node
pop up --suri //Alice

# Deploy to testnet
cargo contract instantiate \
    --suri "your seed phrase" \
    --url wss://rpc.shibuya.astar.network \
    --constructor new \
    --args 42
```

## Testing

### Unit Tests (off-chain)

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[ink::test]
    fn default_works() {
        let contract = MyContract::default();
        assert_eq!(contract.get_value(), 0);
    }

    #[ink::test]
    fn set_value_works() {
        let mut contract = MyContract::new(0);
        assert!(contract.set_value(42).is_ok());
        assert_eq!(contract.get_value(), 42);
    }

    #[ink::test]
    fn deposit_works() {
        let mut contract = MyContract::default();
        // Set caller and value
        let accounts = ink::env::test::default_accounts::<Environment>();
        ink::env::test::set_caller::<Environment>(accounts.alice);
        ink::env::pay_with_call!(contract.deposit(), 1000);
    }
}
```

### E2E Tests (on-chain)

```rust
#[cfg(all(test, feature = "e2e-tests"))]
mod e2e_tests {
    use super::*;
    use ink_e2e::ContractsBackend;

    type E2EResult<T> = std::result::Result<T, Box<dyn std::error::Error>>;

    #[ink_e2e::test]
    async fn it_works<Client: E2EBackend>(mut client: Client) -> E2EResult<()> {
        // Deploy
        let mut constructor = MyContractRef::new(42);
        let contract = client
            .instantiate("my_contract", &ink_e2e::alice(), &mut constructor)
            .submit()
            .await
            .expect("deploy failed");
        let mut call_builder = contract.call_builder::<MyContract>();

        // Call
        let get = call_builder.get_value();
        let result = client.call(&ink_e2e::alice(), &get).dry_run().await?;
        assert_eq!(result.return_value(), 42);

        Ok(())
    }
}
```

## Security Checklist

- [ ] No unbounded iteration — use `Mapping` instead of `Vec` for collections
- [ ] Reentrancy protection — use checks-effects-interactions pattern
- [ ] Integer overflow — use `checked_*` arithmetic or Rust's default overflow checks
- [ ] Access control — verify `self.env().caller()` for privileged operations
- [ ] Don't trust `self.env().transferred_value()` without validation
- [ ] Validate all constructor arguments
- [ ] Test with malicious inputs (zero addresses, max values, empty strings)
