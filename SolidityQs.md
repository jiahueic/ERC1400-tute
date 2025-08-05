# Solidity Questions

### 🧠 1. public vs external

| Keyword    | Can call from inside contract? | Can call from outside (e.g. wallet)? | Gas Cost (if called internally) |
| ---------- | ------------------------------ | ------------------------------------ | ------------------------------- |
| `public`   | ✅ Yes                         | ✅ Yes                               | Slightly more                   |
| `external` | ❌ No                          | ✅ Yes                               | Cheaper                         |

### 🧠 2. pure vs view

| Type   | Reads State? | Modifies State? | Use When…                     |
| ------ | ------------ | --------------- | ----------------------------- |
| `pure` | ❌ No        | ❌ No           | You do **only math or logic** |
| `view` | ✅ Yes       | ❌ No           | You **read from storage**     |

```sh
function add(uint x, uint y) public pure returns (uint) {
  return x + y; // No blockchain reads
}

function getBalance() public view returns (uint) {
  return _balances[msg.sender]; // Reads state
}
```

### 🧠 3. What is calldata?

calldata is used for function parameters that:

- Are read-only
- Come from an external call
- Stay in cheaper, immutable memory

```sh
function setTokenExtension(address extension, string calldata interfaceLabel, ...) external { ... }
```

The string interfaceLabel is passed from the outside, but Solidity doesn’t need to copy it to memory—it just reads it as-is.

## 🧩 Explanation of ERC1400 Code Sections

🔐 allowanceByPartition
Returns how many tokens a spender is allowed to spend from a specific partition of the owner’s balance.

```sh
_allowedByPartition[partition][owner][spender]
```

Think:

"Bob is allowed to use 50 tokens from Alice's UNLOCKED box."

✍️ approveByPartition
Lets Alice say:
"I give Bob permission to spend 50 of my LOCKED tokens."
This updates the \_allowedByPartition mapping and emits an ApprovalByPartition event for off-chain apps to track.

## real world use case for allowance and approval

🔑 Analogy (Simple Terms):
Imagine you give your friend permission to use your prepaid card at a store. You don’t give them your card, but you authorize them to spend up to $100.

✅ Common Real-World Use Cases:
| Use Case | Explanation |
| ---------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Decentralized Exchanges (DEX)** | You approve a DEX contract (e.g., Uniswap) to pull X tokens from your wallet to perform a trade. |
| **Subscription Payments** | You approve a payment contract to deduct a monthly fee (e.g., 10 tokens/month) from your account. |
| **Tokenized Securities (ERC1400)** | You authorize a broker or transfer agent to transfer restricted shares on your behalf. |
| **Lending Platforms** | You approve a lending protocol (e.g., Aave) to take custody of tokens for collateral or repayment. |

🔗 setTokenExtension
Lets the owner attach a smart extension to add logic like:

- KYC verification
- Transfer validation
- Whitelist rules

Think of it like plugging in a USB that adds a new rulebook

🧳 migrate
Used to move to a new version of the token contract.

- Sets the new contract in the ERC1820 registry.
- Optionally disables the current one forever if definitive = true.
