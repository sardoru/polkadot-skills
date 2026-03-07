# Resources — Curated Links

## Official Documentation

| Resource | URL | Use When |
|----------|-----|----------|
| Polkadot Wiki | https://wiki.polkadot.network | General Polkadot concepts, governance, staking |
| Polkadot Developer Docs | https://docs.polkadot.com | Pallet development, runtime configuration |
| PAPI Docs | https://papi.how | Client SDK reference, tutorials |
| ink! Docs | https://use.ink | Smart contract development |
| XCM Docs | https://paritytech.github.io/xcm-docs | Cross-chain messaging |
| Polkadot-SDK | https://github.com/paritytech/polkadot-sdk | Source of truth for all runtime code |

## Tools

| Tool | Purpose | Install |
|------|---------|---------|
| `pop-cli` | Scaffold projects, build contracts, deploy | `cargo install --locked pop-cli` |
| `subxt` | Rust client for Substrate chains | `cargo install subxt-cli` |
| Chopsticks | Fork-based chain testing | `npm i -g @acala-network/chopsticks` |
| Zombienet | Multi-node test networks | `npm i -g @zombienet/cli` |
| Subscan | Block explorer | https://polkadot.subscan.io |
| Polkadot.js Apps | Chain interaction UI | https://polkadot.js.org/apps |
| Subsquare | Governance UI | https://polkadot.subsquare.io |

## Chain RPCs

| Chain | WebSocket | Use |
|-------|-----------|-----|
| Polkadot | `wss://rpc.polkadot.io` | Production relay chain |
| Kusama | `wss://kusama-rpc.polkadot.io` | Canary network |
| Westend | `wss://westend-rpc.polkadot.io` | Testnet (free tokens) |
| Asset Hub (Polkadot) | `wss://polkadot-asset-hub-rpc.polkadot.io` | System parachain for assets |
| Asset Hub (Westend) | `wss://westend-asset-hub-rpc.polkadot.io` | Testnet Asset Hub |

**Prefer Smoldot light client over RPC for production dApps.**

## Key Repositories

| Repo | What's Inside |
|------|--------------|
| `paritytech/polkadot-sdk` | Substrate, FRAME, Cumulus, XCM — the whole stack |
| `polkadot-api/polkadot-api` | PAPI — the modern TypeScript client |
| `paritytech/zombienet` | Integration testing framework |
| `AcalaNetwork/chopsticks` | Fork-based testing |
| `use-ink/ink` | ink! smart contract language |
| `paritytech/subxt` | Rust client library |
| `smol-dot/smoldot` | Light client implementation |
| `paritytech/xcm-docs` | XCM documentation and examples |
| `polkadot-fellows/runtimes` | System chain runtimes |

## Learning Path

### Beginner
1. Read Polkadot Wiki — understand relay chain, parachains, consensus
2. Set up a local dev chain with `pop new parachain`
3. Build a simple pallet with storage + events
4. Connect to it with PAPI

### Intermediate
5. Write ink! contracts and deploy to a testnet
6. Implement XCM transfers between two local chains (Zombienet)
7. Build a React dApp with wallet connection
8. Submit a governance proposal on Westend

### Advanced
9. Implement custom XCM instructions
10. Build a coretime-powered Agent Space
11. Contribute to Polkadot SDK
12. Prepare for JAM integration

## Testnet Faucets

| Network | Faucet |
|---------|--------|
| Westend | https://faucet.polkadot.io/westend |
| Rococo | https://faucet.polkadot.io/rococo |
| Paseo | https://faucet.polkadot.io/paseo |

## Community

| Platform | Link |
|----------|------|
| Polkadot Forum | https://forum.polkadot.network |
| Substrate Stack Exchange | https://substrate.stackexchange.com |
| Polkadot Discord | https://dot.li/discord |
| Element (Matrix) | https://matrix.to/#/#polkadot-watercooler:parity.io |
