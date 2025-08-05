# How partition transfers work in ERC1400 and linking to real-world use-case

Let’s say:

Alice has 100 tokens, split into two boxes (partitions):

"LOCKED" box: 60 tokens

"UNLOCKED" box: 40 tokens

Bob wants to get 20 tokens

### 🎯 ERC1400 makes you specify which partition the tokens come from.

The function you’d call is:

```sh
transferByPartition(partitionLabel, toAddress, amount, data)
```

Example:

```sh
transferByPartition("UNLOCKED", Bob, 20, "")
```

So:

Tokens only come from the "UNLOCKED" partition

It won't touch tokens in the "LOCKED" partition

The system can also check: “Are you allowed to move these?”

💡 Why this matters in real-world use
Let’s say a company issues tokenized shares:
| Partition | What it means |
| --------------- | ----------------------------------------- |
| `LOCKED` | Shares under 1-year lockup (can’t sell) |
| `UNLOCKED` | Shares you can trade freely |
| `KYC_INVESTOR` | Shares only for KYC-verified investors |
| `DIVIDEND_2025` | Special tokens eligible for 2025 dividend |
Each group needs different rules, and regulators care.

💼 Admins can also force a transfer (e.g. for legal or compliance reasons):

```sh
controllerTransferByPartition(partition, from, to, amount, ...)
```

This is useful for:

- Freezing assets
- Enforcing court orders
- Correcting mistakes

## 🔡 PART 2: Why partitions are bytes32

✅ 1. Efficient

- bytes32 is fixed-size = 32 bytes
- Cheaper to store and compare than string
- Gas-efficient and faster lookup (in mappings)

✅ 2. Flexible

- You can store anything:
  - "LOCKED" → 0x4c4f434b45440000000000000000000000000000000000000000000000000000
  - "UNLOCKED"→ similar

✅ 3. Secure & precise

- You avoid issues with string typos or encoding bugs
- You can define enums manually using hashes or constants

🧠 Tip: Convert between string and bytes32

```sh
bytes32 label = bytes32("LOCKED"); // pad with zeroes
```
