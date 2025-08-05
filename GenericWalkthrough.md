# ERC1400 Generic Walkthrough

### State: balances and partitions

```sh
mapping(address => mapping(bytes32 => uint256)) internal \_balanceOfByPartition;
mapping(address => uint256) internal \_balances;
```

- Meaning: For each person, we track:

  - How many tokens in each partition (locked/unlocked).
  - Their total balance too.

### Total supply and partitions

```sh
bytes32[] internal _totalPartitions;
mapping(bytes32 => uint256) internal _totalSupplyByPartition;
```

Meaning: The token knows all the different partition labels used, and how many tokens exist in each partition overall.

### Issue tokens (mint)

```sh
function issueByPartition(bytes32 partition, address tokenHolder, uint256 value, bytes calldata data) external { … }
```

Meaning: Creating new tokens inside a specific partition for someone. For example: give 100 locked tokens to Alice.

### Transfer by partition

```sh
function transferByPartition(bytes32 partition, address to, uint256 value, bytes calldata data) external returns (bytes32) { … }
```

Meaning: Move tokens from your partition box to someone else’s, but only within that partition. Eg: 10 unlocked tokens from Alice to Bob.

### Transfer checks

```sh
_canTransferByPartition(...)
```

Purpose: Checks if move is allowed. If not, it returns a reason like “not authorized” or “locked.”

### Forced transfers

```sh
function controllerTransfer(...) external {
    // moves tokens even if holder doesn’t consent
}
```

Meaning: A special admin can move tokens for someone else when law demands it.

### ERC‑20 compatibility

```sh
function transfer(address to, uint256 value) external returns (bool) { … }
```

Meaning: Allows standard transfer ignoring partitions—the basic coin move for simple apps.

### Document features

```sh
function setDocument(...) external { … }
```

What it does: Attach legal docs or ID info linked to the token—like saying “this token represents Share Certificate #123.”
