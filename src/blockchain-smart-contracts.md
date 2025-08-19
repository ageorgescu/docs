## ⛓️ Blockchain & Smart Contracts Deep-Dive (Ethereum-focused)

### Public Ethereum Basics

- **Accounts**
  - EOAs (Externally Owned Accounts) control addresses via private keys.
  - Contract Accounts store bytecode and persistent storage.
- **Transactions**
  - Sent by EOAs, can transfer ETH or trigger contract code.
  - Fields: `nonce`, `to`, `value`, `data`, `gasLimit`, `gasPrice`.
  - Must be signed using ECDSA on curve secp256k1.
- **Gas**
  - Fee paid by sender: `gasUsed * gasPrice`.
  - Out-of-gas causes a revert without persistent state change.

---

### EVM Architecture

| Component | Description                          |
| --------- | ------------------------------------ |
| Stack     | LIFO (1024 × 256-bit items)          |
| Memory    | Byte-addressable, temporary per txn  |
| Storage   | Persistent key/value (32-byte slots) |
| Calldata  | Read-only input data for functions   |

---

### Solidity Concepts

- **Visibility** ⇒ `public`, `external`, `internal`, `private`
- **Functions**: `view`(reads), `pure`(no read), `payable`(receives ETH)
- State variables packed sequentially into 32-byte slots → pack small types for gas savings.
- Example structure:

```solidity
uint public value;
function set(uint v) public { value = v; }
function get() external view returns(uint) { return value; }
```

---

### Proxy / Upgradeability Patterns

| Pattern           | How it works                         | Notes                                 |
| ----------------- | ------------------------------------ | ------------------------------------- |
| Transparent Proxy | Proxy delegates to Logic contract    | Admin calls go through admin slot     |
| UUPS (ERC-1967)   | Logic contains own upgrade function  | Cheaper but upgrade fn must be secure |
| delegatecall risk | Logic code executes in proxy storage | Must only delegate trusted code       |

---

### Signature Verification

- Off-chain sign → on-chain verify using `ecrecover(hash, v, r, s)`.
- Ethereum uses **Keccak-256**, not SHA-256.
- Signature params:  
  - `r`, `s`: signature points  
  - `v`: recovery id

---

### Common Vulnerabilities

| Risk               | Description                                    | Mitigation                                           |
| ------------------ | ---------------------------------------------- | ---------------------------------------------------- |
| Re-entrancy        | External call before state update              | checks-effects-interactions, mutex, reentrancy guard |
| delegatecall       | Executes in caller context → overwrite storage | Only delegate to trusted logic contracts             |
| tx.origin          | Can be exploited via phishing contract calls   | Use `msg.sender` instead                             |
| Overflow (pre-0.8) | Wraps around on overflow                       | Use Solidity ≥0.8 or SafeMath library                |
| selfdestruct       | Removes code from storage                      | Don’t expose `selfdestruct` in production logic      |

---

### Gas Optimization Tips

- Use `uint256` consistently
- Mark constants as `immutable` / `constant`
- Pack multiple `uint8/bool` into one slot
- Avoid unnecessary storage writes

---

### Frequently Asked Interview Questions

**Q: How does `delegatecall` differ from `call` and `staticcall`?**  
A: `call` invokes another contract and executes code in the *callee’s* storage context. `delegatecall` executes the callee’s code in the *caller’s storage* — which is how proxy upgrade patterns work but also how malicious libraries can overwrite your state. `staticcall` is like `call` but read-only and reverts if state changes are attempted.

---

**Q: Describe the lifecycle of a transaction in Ethereum.**  
A: A user signs the tx offline with their private key → broadcasts to a node → enters the **mempool**. Miners pick a batch from mempool, order them by gas price, execute in EVM changing state, and include them in a block. Block is propagated and eventually finalised when many confirmations are added. Tx includes `nonce` for replay protection.

---

**Q: What is a nonce and why is it needed?**  
A: Each EOA has a nonce that increments with every transaction sent. It ensures transactions cannot be replayed (prevents double-spend) and establishes ordering. If a tx has an old or duplicate nonce it is rejected.

---

**Q: How is `mapping(address => uint)` stored in storage?**  
A: Storage is a key-value store using 32-byte slots. Each mapping entry is stored at `keccak256(h(k) . p)` where `p` is position of the mapping in storage and `k` is the key. So even though a mapping looks like one variable, internally each entry is hashed to its own storage slot.

---

**Q: What are proxy upgrade patterns and what are their risks?**  
A: Proxies use `delegatecall` to forward calls to a logic contract. The logic can be upgraded while the proxy retains state. Types include Transparent Proxy and UUPS. Risks include mistakes such as storage layout mismatch (variables in new logic don’t align), or bad upgrade functions allowing malicious takeover.

---

**Q: Show & fix a re-entrancy attack**  
Example attack: contract sends Ether *before* updating its state; attacker repeatedly re-enters fallback to drain balance.  
**Fix**: use Checks-Effects-Interactions pattern (update state *before* transferring Ether), or Guard with `nonReentrant` modifier.

---

