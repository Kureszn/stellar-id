# StellarID

A decentralized identity and verifiable credentials contract on Stellar Soroban. Issuers attach on-chain credentials to wallet addresses — KYC status, merchant verification, accredited investor status — that any dApp can query with a single function call.

[![CI](https://github.com/laugh-tales/stellar-id/actions/workflows/ci.yml/badge.svg)](https://github.com/laugh-tales/stellar-id/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## What It Does

StellarID provides shared identity infrastructure for the Stellar ecosystem. Instead of every DeFi protocol or anchor building its own KYC system, StellarID offers a single reusable layer:

- **Issuers** (anchors, fintechs, regulators) register and issue credentials to wallet addresses
- **Subjects** collect credentials from multiple issuers into a single on-chain identity profile
- **Consumers** (dApps, contracts) call `has_valid_credential()` to gate access in one line

**Use cases:**
- DeFi protocols gating access to verified users
- Anchors attesting KYC completion before enabling fiat on/off ramps
- DAOs verifying accredited investor status
- Marketplaces verifying merchant identity
- Cross-protocol credential sharing without repeated KYC

## Contract Functions

| Function | Who Calls It | Description |
|---|---|---|
| `initialize(admin)` | Deployer | Set up the contract |
| `register_issuer(admin, issuer, name, trust_level)` | Admin | Approve a new credential issuer |
| `deactivate_issuer(admin, issuer)` | Admin | Disable an issuer |
| `authorize_sub_issuer(parent, sub_issuer)` | Issuer | Delegate issuance to a sub-issuer |
| `revoke_sub_issuer(parent, sub_issuer)` | Issuer | Remove sub-issuer delegation |
| `register_schema(issuer, name, description)` | Issuer | Define a new credential type |
| `issue_credential(issuer, subject, schema_id, duration)` | Issuer | Issue a credential to a wallet |
| `revoke_credential(issuer, credential_id)` | Issuer | Revoke an issued credential |
| `has_valid_credential(subject, schema_id)` | Anyone | Check if subject has active credential |
| `has_credential_from_issuer(subject, issuer)` | Anyone | Check if subject was credentialed by issuer |
| `get_credential(credential_id)` | Anyone | Get credential details |
| `get_identity(subject)` | Anyone | Get identity profile and reputation score |
| `get_issuer(issuer)` | Anyone | Get issuer details |
| `get_schema(schema_id)` | Anyone | Get schema details |
| `get_subject_credentials(subject)` | Anyone | List all credential IDs for a subject |
| `get_credential_count()` | Anyone | Total credentials issued |
| `get_schema_count()` | Anyone | Total schemas registered |
| `is_sub_issuer(parent, sub_issuer)` | Anyone | Check sub-issuer authorization |

## Identity Lifecycle

```
Admin           Issuer              Subject              Consumer dApp
  |                |                   |                      |
  |-- register_issuer() --> |           |                      |
  |                |                   |                      |
  |                |-- register_schema() -->                   |
  |                |   (schema stored on-chain)               |
  |                |                   |                      |
  |                |-- issue_credential() --> |               |
  |                |   (credential + identity created)        |
  |                |                   |                      |
  |                |                   |  <-- has_valid_credential()
  |                |                   |      --> true/false
```

## Reputation Score

Each identity has a `reputation_score` derived from credential count and issuer trust level. It increases as more trusted issuers add credentials. The score caps at 1000.

```
reputation = min(credential_count × 10 + trust_bonus, 1000)
```

## Build

**Prerequisites:**
- Rust 1.75+
- Soroban CLI ([install guide](https://developers.stellar.org/docs/smart-contracts/getting-started/setup))

```bash
# Install WASM target
rustup target add wasm32-unknown-unknown

# Clone and run tests
git clone https://github.com/laugh-tales/stellar-id
cd stellar-id
make test

# Build WASM
make build
```

## Deploy to Testnet

```bash
# Generate a testnet identity
soroban keys generate --global deployer --network testnet
soroban keys fund deployer --network testnet

# Deploy
soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/stellar_id.wasm \
  --network testnet \
  --source deployer

# Initialize
soroban contract invoke \
  --id CONTRACT_ID \
  --network testnet \
  --source deployer \
  -- initialize \
  --admin $(soroban keys address deployer)
```

See [docs/DEPLOYMENT.md](./docs/DEPLOYMENT.md) for the full walkthrough.

## Project Structure

```
stellar-id/
├── contracts/
│   └── stellar-id/
│       ├── src/
│       │   └── lib.rs       # Contract + 15 tests
│       └── Cargo.toml
├── docs/
│   ├── ARCHITECTURE.md      # System design and data model
│   ├── DEPLOYMENT.md        # Testnet and mainnet guide
│   └── INTEGRATION.md       # How to integrate has_valid_credential
├── scripts/
│   └── deploy.sh            # Deploy to testnet
├── .github/
│   └── workflows/
│       └── ci.yml           # Test + lint + WASM build
├── Makefile
├── rust-toolchain.toml
├── CONTRIBUTING.md
├── SECURITY.md
└── Cargo.toml
```

## Contributing

This project participates in the [Stellar Wave Program](https://drips.network/wave/stellar) on Drips. Contributors earn USDC rewards for resolving issues.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for setup and contribution guidelines.

Issues are labeled by complexity:
- 🟢 `good first issue` — Tests, docs, small fixes (100 pts)
- 🟡 `enhancement` — New features (150 pts)
- 🔴 `high complexity` — Architecture changes (200 pts)

## License

MIT — see [LICENSE](./LICENSE) for details.
