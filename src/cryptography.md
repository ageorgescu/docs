## 🔐 Cryptography Deep-Dive (Interview-Focused)

### Hashing

- One-way function → deterministic, irreversible.
- Used for integrity checking, digital signatures, Merkle trees.
- Examples:
  - SHA-256 (Bitcoin)
  - Keccak-256 (Ethereum)
- Properties:
  - **Pre-image resistance** – cannot find input from hash.
  - **Collision resistance** – cannot find two inputs with same hash.
  - **Avalanche effect** – small change → large hash difference.

---

### Symmetric Encryption

- Same key for encryption & decryption.
- Very fast → used for bulk data.
- Examples:
  - AES (Rijndael) in CBC / GCM modes
  - ChaCha20-Poly1305 (mobile performance)
- Requires **key exchange** separately.

---

### Asymmetric Encryption

- **Public key** encrypts → **private key** decrypts.
- Enables secure key exchange & digital signatures.
- Examples:
  - RSA (slow, large key sizes)
  - Elliptic-Curve (ECDSA, ECDH) → shorter keys, faster
  - Ethereum uses **ECDSA** on curve `secp256k1`.

---

### Digital Signatures (Ethereum flow)

1. Hash message (Keccak-256).
2. Sign with private key ⇒ `r`, `s`, `v`.
3. On-chain, use `ecrecover(hash, v, r, s)` to recover signer’s address.
4. Signature must satisfy `s <= secp256k1n/2` to be canonical.

---

### Key Derivation

| Use case         | Algorithm                                |
| ---------------- | ---------------------------------------- |
| Password hashing | PBKDF2, Argon2, scrypt                   |
| Wallets          | BIP-39 mnemonics → seed → BIP-32 HD keys |
| Session keys     | HKDF                                     |

---

### Merkle Trees

- Tree built from hashed leaf data → parents = hash(left + right).
- Enables efficient proof of inclusion (**O(log N)**).
- Used in:
  - Blockchain block headers (`MerkleRoot`)
  - Sparse Merkle tries (Ethereum state trie)

---

### Frequently Asked Interview Questions

**Q: What is the difference between hashing and encryption?**  
A: Hashing is a one-way operation: you cannot recover the input from the output. It’s used for integrity checks and storing passwords (e.g., SHA-256, Keccak-256). Encryption is reversible: data is transformed using a key and can be decrypted back using the key (symmetric) or key pair (asymmetric).

---

**Q: Explain how an ECDSA signature works and how Ethereum verifies it.**  
A: A message is hashed (Keccak-256), then the signer uses their private key on the elliptic-curve algorithm to produce (`r`, `s`, `v`). To verify, Ethereum uses `ecrecover(hash, v, r, s)` to recover the signer’s address from the signature, then checks it matches the expected signer.

---

**Q: Why are elliptic curves preferred over RSA in blockchains?**  
A: Elliptic-curve cryptography provides the same level of security as RSA but with much smaller keys (e.g., 256-bit ECC ≈ 3072-bit RSA), resulting in faster computation, lower storage, and less network overhead — critical for blockchain nodes.

---

**Q: Why use PBKDF2 / bcrypt / scrypt instead of plain SHA-256 for passwords?**  
A: These are *key-derivation functions* that intentionally slow down hashing (using multiple rounds and memory hardness). This makes brute-force attacks far more expensive than a fast function like SHA-256, which is too quick and vulnerable to GPU-based cracking.

---

**Q: What is a Merkle proof and why is it efficient?**  
A: A Merkle proof is used to verify that a specific leaf is included in a Merkle tree by providing a path of sibling hashes from leaf to root. The verifier only needs O(log N) hashes instead of all leaves, making proof size and verification efficient (used in blockchain block headers, Sparse Merkle Trees).

---

**Q: What’s the difference between symmetric and asymmetric keys?**  
A: Symmetric encryption uses a single shared key for both encryption and decryption (e.g., AES). Asymmetric encryption uses separate **public** and **private** keys (e.g., RSA, ECDSA) where the public key encrypts/verifies and private key decrypts/signs. Asymmetric is useful when sharing a key securely is difficult.

---
