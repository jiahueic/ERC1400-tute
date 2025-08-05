# Resources
```sh
https://github.com/Consensys/UniversalToken/blob/master/contracts/ERC1400.sol
https://github.com/SecurityTokenStandard/EIP-Spec/blob/master/eip/eip-1400.md
https://youtu.be/TegyVjk8ejM?si=IkfvGIloI62hbx5U
```
# FAQ
### 🧱 Clarifying: Can there be multiple tokens in each partition?
✅ Yes. Each partition can contain many tokens.
Think of partitions like labeled boxes:
- Each box has a label: LOCKED, UNLOCKED, SERIES_A, etc.
- Inside the box, there can be many tokens (bricks).
- Each token in the same box behaves the same way.

🧠 Why group tokens like this?
Because not all tokens are equal when we talk about real-world assets like:

Shares issued at different times (Series A vs Series B)

Some tokens might be locked for 1 year

Others can be transferred freely

Each partition:
Holds many tokens that follow the same rule

Is like a sub-balance of your total

🎨 Visual Example:
Let's say Alice has 100 tokens.
They are split like this:
| Partition  | Tokens (Bricks) |
| ---------- | --------------- |
| `LOCKED`   | 60              |
| `UNLOCKED` | 40              |
Alice owns all 100 tokens, but they are grouped into boxes with different rules.
🧩 In code
In the ERC1400 code, this is represented like:
```sh
mapping(address => mapping(bytes32 => uint256)) internal _balanceOfByPartition;
```
This says:

For each person (address) and each box (bytes32 partition), how many tokens (uint256) do they have?
```sh
_balanceOfByPartition[Alice]["LOCKED"] = 60;
_balanceOfByPartition[Alice]["UNLOCKED"] = 40;
```
