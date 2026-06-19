# Deployment Guide

## Prerequisites

- Rust 1.75+ with `wasm32-unknown-unknown` target
- Soroban CLI installed
- A funded Stellar testnet account

## 1 — Install Tools

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-unknown-unknown

# Install Soroban CLI
cargo install --locked soroban-cli
```

## 2 — Build the Contract

```bash
git clone https://github.com/laugh-tales/stellar-id
cd stellar-id
make build
# WASM output: target/wasm32-unknown-unknown/release/stellar_id.wasm
```

## 3 — Configure Testnet Identity

```bash
# Generate keypair
soroban keys generate --global deployer --network testnet

# Fund from friendbot
soroban keys fund deployer --network testnet

# Confirm balance
soroban keys address deployer
```

## 4 — Deploy

```bash
CONTRACT_ID=$(soroban contract deploy \
  --wasm target/wasm32-unknown-unknown/release/stellar_id.wasm \
  --network testnet \
  --source deployer)

echo "Deployed: $CONTRACT_ID"
```

Or use the deploy script:
```bash
bash scripts/deploy.sh
```

## 5 — Initialize

```bash
soroban contract invoke \
  --id "$CONTRACT_ID" \
  --network testnet \
  --source deployer \
  -- initialize \
  --admin $(soroban keys address deployer)
```

## 6 — Register an Issuer

```bash
soroban contract invoke \
  --id "$CONTRACT_ID" \
  --network testnet \
  --source deployer \
  -- register_issuer \
  --admin $(soroban keys address deployer) \
  --issuer ISSUER_ADDRESS \
  --name '"MyAnchor KYC"' \
  --trust_level 80
```

## 7 — Register a Schema

```bash
soroban contract invoke \
  --id "$CONTRACT_ID" \
  --network testnet \
  --source myissuer \
  -- register_schema \
  --issuer $(soroban keys address myissuer) \
  --name '"KYC Verified"' \
  --description '"Basic identity verification"'
```

## 8 — Issue a Credential

```bash
soroban contract invoke \
  --id "$CONTRACT_ID" \
  --network testnet \
  --source myissuer \
  -- issue_credential \
  --issuer $(soroban keys address myissuer) \
  --subject SUBJECT_WALLET_ADDRESS \
  --schema_id 1 \
  --duration_seconds 0
```

`duration_seconds = 0` means the credential does not expire.

## 9 — Verify a Credential

```bash
soroban contract invoke \
  --id "$CONTRACT_ID" \
  --network testnet \
  -- has_valid_credential \
  --subject SUBJECT_WALLET_ADDRESS \
  --schema_id 1
```

Returns `true` or `false`.

## Mainnet Notes

- Complete a full security audit before mainnet deployment
- Set a multi-sig or hardware wallet as the admin address
- Use appropriate trust levels — only well-known anchors should get 80+
- Monitor the `credential_issued` and `credential_revoked` event stream
