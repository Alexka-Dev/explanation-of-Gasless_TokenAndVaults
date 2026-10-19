# Gasless Token & Vault (ERC-2612 & EIP-712)

A Foundry-based smart contract project demonstrating gasless transactions on Ethereum using **ERC-2612 Permits** and custom **EIP-712 Typed Data Signatures**.

This repository contains two core smart contracts:
1. **`PermitToken.sol`**: An ERC-20 token implementing standard ERC-2612 gasless approvals.
2. **`GaslessVault.sol`**: A vault supporting gasless deposits (combining `permit` + `transferFrom` in a single transaction) and custom off-chain signed authorizations for gasless withdrawals.

---

## 🌟 Overview

In traditional ERC-20 interactions, users need two separate on-chain transactions to interact with a smart contract vault:
1. `approve(spender, amount)` (Costs Gas)
2. `transferFrom(owner, recipient, amount)` (Costs Gas)

This architecture eliminates the need for prior on-chain approvals and allows meta-transactions (relayers paying gas on behalf of users):
- **Gasless Deposit**: Leverages `ERC20Permit` to perform approval and deposit in a single transaction.
- **Gasless Withdrawal**: Leverages a custom EIP-712 typed struct (`WithdrawAuthorization`) to authorize withdrawals via off-chain signatures.

---

## 🏗 Architecture & Contracts

### 1. `PermitToken.sol`
* **Authors**: Alexka Lovera
* **Solidity Version**: `0.8.37`
* **Dependencies**: OpenZeppelin `ERC20`, `ERC20Permit`

An ERC-20 token contract that standardizes gasless approvals via [ERC-2612](https://eips.ethereum.org/EIPS/eip-2612).

#### Key Functionality:
- `permit(owner, spender, value, deadline, v, r, s)`: Sets the allowance for `spender` using an off-chain signature signed by `owner`.

---

### 2. `GaslessVault.sol`
* **Authors**: Alexka Lovera
* **Solidity Version**: `0.8.37`
* **Dependencies**: OpenZeppelin `EIP712`, `ECDSA`, `Nonces`

A vault contract designed to work with `PermitToken`. It allows users to deposit and withdraw tokens both traditionally and gaslessly.

#### Core Functions:
* `deposit(uint256 amount)`: Standard deposit (requires prior `approve`).
* `depositWithPermit(address owner, uint256 amount, uint256 deadline, uint8 v, bytes32 r, bytes32 s)`: Executes `permit()` and transfers tokens to the vault in **one atomic transaction**.
* `withdraw(address to, uint256 amount)`: Standard withdrawal of vault balance.
* `withdrawBySig(address owner, address to, uint256 amount, uint256 deadline, uint8 v, bytes32 r, bytes32 s)`: Withdraws tokens on behalf of `owner` using a custom signed `WithdrawAuthorization` message.

---

## 🔐 EIP-712 & Replay Protection

To ensure signatures cannot be reused or replayed across different chains or contracts, both contracts use OpenZeppelin's `EIP712` and `Nonces` utilities.

### Custom Struct Hash for Withdrawals
The `GaslessVault` defines a custom typehash for user authorization:

```solidity
bytes32 public constant WITHDRAW_TYPEHASH = keccak256(
    "WithdrawAuthorization(address owner,address to,uint256 amount,uint256 nonce,uint256 deadline)"
);
```

### Protection Mechanisms:

1. **Domain Separators**: Each contract computes a unique domain separator based on contract name, version, chain ID, and contract address.
2. **Nonces**: Monotonically increasing per-account nonces managed via `_useNonce(owner)` prevent replay attacks.
3. **Deadlines**: Signatures expire after the specified `deadline` timestamp.

---

## 🚀 Getting Started

### Prerequisites

Ensure you have [Foundry](https://getfoundry.sh/) installed:

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup
```

### Installation

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/Alexka-Dev/explanation-of-Gasless_TokenAndVaults](https://github.com/Alexka-Dev/explanation-of-Gasless_TokenAndVaults)
   cd gasless-vault
   ```

2. **Install dependencies:**
   ```bash
   forge install OpenZeppelin/openzeppelin-contracts
   ```

3. **Build the project:**
   ```bash
   forge build
   ```

---

## 🧪 Testing

The repository comes with a comprehensive Foundry unit test suite.

### Running Tests

Execute all tests:
```bash
forge test
```

Run tests with increased verbosity to view logs and execution traces:
```bash
forge test -vvvv
```

Run specific tests (e.g., testing gasless withdrawal):
```bash
forge test --match-test test_GaslessWithdraw_Success
```

---

## 📄 License

This project is licensed under the **MIT License**.

## Author.

Alexka Lovera
