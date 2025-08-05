# Modifiers in ERC1400

### 🔎 Key Modifiers Used in ERC1400

1. onlyOwner

```sh
modifier onlyOwner() {
    require(msg.sender == _owner, "Only owner can do this");
    _;
}
```

Used in functions like:

```sh
function setDocument(...) public onlyOwner { ... }
```

🔒 Purpose: Ensures only the token creator/owner can manage documents or control settings.

2. isControllable

```sh
function controllerTransferByPartition(...) external isControllable { ... }
```

🔒 Purpose: Prevents forced transfers unless controllable mode is enabled.
🎮 Who enables it?
The token contract owner sets this flag when the contract is created:

```sh
_isControllable = true;
```

It can’t be turned off afterward in the UniversalToken example.
🧠 When to use:
✅ Use isControllable = true when you're handling regulated tokens (e.g. shares, bonds, legal assets).
❌ Avoid it for totally decentralized tokens.

3. isIssuable

```sh
modifier isIssuable() {
    require(_isIssuable, "Token not issuable");
    _;
}

```

Used in:

```sh
function issueByPartition(...) public isIssuable { ... }
```

🔒 Purpose: Disables minting if tokens are finalized. Useful for regulated assets (e.g. once a fund closes, no new shares can be created).
🎮 Who sets it?
Only the owner (the one who deployed the token contract) can flip the switch.

```sh
function renounceIssuance() external onlyOwner {
  _isIssuable = false;
}
```

🧠 When to use:
✅ Use isIssuable = true at the beginning when you're distributing tokens
❌ Set it to false when you’re done issuing — e.g. after a fundraising round ends or shares are finalized.
Once it's false, no one (not even the owner) can issue new tokens. Forever!

### Q: Is it possible to renounce issuance just for a partition

🤔 No, ERC1400 does not natively support renouncing issuance per partition — the isIssuable flag is global, meaning it applies to the entire token, not individual partitions.
🧱 Simple Example:
Let’s say your token has:

- UNLOCKED partition → for public tokens
- LOCKED partition → for investor tokens
  If you call:

```sh
renounceIssuance()
```

That disables minting for all partitions, not just LOCKED or UNLOCKED.

🛠 Could we add it?
Yes! You could modify the contract like this:

```sh
mapping(bytes32 => bool) public isPartitionIssuable;
```

Then check this in issueByPartition:

```sh
require(isPartitionIssuable[partition], "Partition not issuable");
```

✅ This would give you fine-grained control, useful in multi-phase offerings or hybrid securities.
