# ERC1400 in Production

🧠 What’s Going On?
You want to track and report on movements and balances of a special kind of Ethereum token called ERC-1400. These are used for financial securities and are a bit more complex than normal tokens because they are partitioned – think of it like having separate pockets in one wallet.
To do this tracking, you:

1. Deploy the ERC-1400 smart contract (the token lives here).
2. Create a listener to catch events when something happens (issue, transfer, redeem tokens).
3. Run an indexing task whenever such events happen, to update a database-like structure (called the asset manager) with that info.

## 🔗 System Components Overview (ERC-1400 with FireFly)

| Component             | Role                                                                                                                          |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **ERC-1400 Contract** | Smart contract representing tokenized securities. Emits key events like `IssuedByPartition` during transfers or issuances.    |
| **Blockchain Events** | Logs emitted by the blockchain when certain actions happen (e.g. token issuance, transfer). Can be listened to in real-time.  |
| **FireFly Listener**  | Watches for blockchain events. Triggers actions when specific ERC-1400 events occur (e.g. listens for `TransferByPartition`). |
| **Indexer Task**      | Backend service that runs when an event is detected. Updates pools, token balances, partition data in FireFly’s database.     |
| **Asset Manager**     | Manages ownership records. Think of it as a smart, real-time spreadsheet showing token distribution by owner and partition.   |

➡️ So when you issue/transfer/redeem tokens, your contract emits an event, the listener hears it, and the indexer task updates the asset manager.

## 👶 Code Explanation Like You're 5

Now let’s explain the main parts of the code step by step, like you’re five
🔤 Step 1: Naming Things

```sh
- name: naming
  type: jsonata_template
  options:
    template: |-
      {
        "partition": ...,
        "pool": ...,
        "activity": ...,
        "protocolIdSafe": ...
      }
```

💡 What this does: It figures out what to call the “pocket” (partition), and sets up names so we can track what’s going on. It checks whether the event uses \_partition or \_fromPartition, and saves the name of the pool and activity.
🧸 Think of it like this: "Oh! A toy was put into the blue basket. Let’s write down that it went to the blue basket!"

🧱 Step 2: Set Up Common Info

```sh
- name: common_upsert
  type: jsonata_template
  options:
    template: { ... }
```

💡 What this does: It creates entries for:

The token contract address

The activity (something happened!)

The pool (which partition the token went into)

🧸 Like saying: "We saw something happen with this toy box. If we haven’t seen this toy box before, write it down

✨ Step 3: Handle Issuing Tokens

```sh
- name: issue_upsert
  type: data_model_update
  skip: input.blockchainEvent.name != "IssuedByPartition"
  ...
```

💡 What this does:

Adds the recipient's address

Adds a transfer entry: how many tokens were sent, where, to whom

Logs an event in the system
🧸 Like saying: "Someone put 10 apples in the blue basket for Alice. Let’s remember that!"

🔁 Step 4: Handle Transfers

```sh
- name: transfer_upsert
  type: data_model_update
  skip: input.blockchainEvent.name != "TransferByPartition"
  ...
```

💡 What this does:

- Notes who sent and received tokens
- Logs the token movement between addresses in a pool

❌ Step 5: Handle Redemptions

- name: redeem_upsert
  type: data_model_update
  skip: input.blockchainEvent.name != "RedeemedByPartition"
  ...
  💡 What this does:

Notes who burned (redeemed) their tokens

Updates the balance and logs the event

🔗 Step 6: Connect Pool to Asset

```sh
- name: link_asset
  type: data_model_update
  ...
```

💡 What this does:

It says “this pool belongs to this asset”
🧸 Like saying: "This blue basket is part of the toy collection called 'ERC1400-TokenX'."

✅ Part 1: Difference Between Pool and Asset (ERC-1400 + FireFly Context)
When working with ERC-1400 tokens in FireFly, the concepts of "pool" and "asset" help model partitioned tokens and how they're grouped.

- Asset

  - Think of an asset as the main token or financial instrument.

  - One ERC-1400 smart contract = one asset.

  - Example: “ABC Corp Bond 2025” is an asset.

Pool

- A pool is a partition or subdivision of the asset.

- Each pool tracks balances for a specific partition within the asset.

- Example: If “ABC Corp Bond 2025” (the asset) has:

  Class A (partition 0xAAA)

  Class B (partition 0xBBB)
  Then:

  Pool 1 = “Partition 0xAAA” of that asset

  Pool 2 = “Partition 0xBBB” of that asset

🧠 Why is this distinction important?
ERC-1400 supports partially fungible tokens — some partitions might represent different rights (like voting vs non-voting shares). FireFly models this by grouping related pools under a single asset.
