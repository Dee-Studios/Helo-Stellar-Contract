## Helo-Stellar-Contract

# Helo — Contract

This Soroban smart contract is setup to power up Helo's immutable payment audit trail. 

It is written in Rust and deployed on the Stellar network.

---

## What is our project about:

The `Helo-Stellar-Contract` repo is the on-chain core of Helo. 

Everything else — the backend, the frontend, the approval workflow — exists to feed verified, multi-signed payment records into this contract.

Once a payment is recorded here, it cannot be changed, deleted, or disputed. The contract stores who approved it, when, how much, and to whom. 

That record is what finance teams hand to auditors instead of a spreadsheet.

The contract is intentionally small. It does not manage funds, it does not move money. It is a permanent, publicly verifiable ledger of payment decisions that were already authorised off-chain by real wallet holders.

---

## How Soroban contracts work

Soroban is Stellar's smart contract platform. Contracts are written in Rust and compiled to WebAssembly (WASM), then deployed to the Stellar network. Once deployed, a contract has its own address and its own persistent storage — data written to it lives on-chain indefinitely.

Any account can call a contract's public functions by submitting a Stellar transaction. The contract validates inputs, enforces its own rules, and writes to storage. The result is permanently recorded in the ledger.

Helo's contract exposes three functions: `initialize`, `record_payment`, and `verify_payment`.

---

## Tech

| Tool | Why |
|------|-----|
| Rust | Contract language — same language as the backend |
| `soroban-sdk` | Stellar's official smart contract SDK |
| `wasm32-unknown-unknown` | Compile target — contracts run as WASM on-chain |
| Stellar CLI | Build, test, deploy, and invoke contracts locally and on testnet |

---

## Prerequisites

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Add the WASM compile target
rustup target add wasm32-unknown-unknown

# Install Stellar CLI
cargo install --locked stellar-cli --features opt
```

---

## Setup and build

```bash
git clone https://github.com/yourname/helo-contract
cd helo-contract

# Compile the contract to WASM
stellar contract build

# Compiled output
# target/wasm32-unknown-unknown/release/helo_contract.wasm
```

---

## Contract functions

##### `initialize(admin, required_approvals)`

Called once at deployment. Sets the admin address (your backend's keypair) and how many wallet signatures are required before a payment can be recorded. Only the admin can call `record_payment`.

##### `record_payment(payment_id, amount_usdc, recipient, purpose, approvers, tx_hash)`

Called by the backend after enough signatures have been collected. Writes a `PaymentRecord` to persistent contract storage. Emits an event so external services can listen for new records. This function requires admin authorisation — it cannot be called by anyone else.

##### `verify_payment(payment_id)`

Open to anyone. Returns the full `PaymentRecord` for a given payment ID, or nothing if it does not exist. This is what the audit proof export calls — no trust required, anyone can independently verify a payment happened.

---

## Data stored on-chain

Every recorded payment stores the following permanently:

```
payment_id     — unique identifier matching the backend database
amount_usdc    — payment amount in stroops (1 USDC = 10,000,000 stroops)
recipient      — Stellar wallet address of the recipient
purpose        — human-readable description of the payment
approvers      — list of wallet addresses that signed the approval
tx_hash        — Stellar transaction hash for independent verification
timestamp      — ledger timestamp at the moment of recording
```

This data lives in Soroban's **persistent storage** — it survives ledger upgrades and is never automatically cleared.

---

## Project structure

```
src/
  lib.rs        # Contract entry point — public functions
  types.rs      # PaymentRecord struct
  storage.rs    # Read/write helpers for contract storage
  errors.rs     # ContractError definitions
Cargo.toml
```

