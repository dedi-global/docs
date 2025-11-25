# How are DeDi.global entries cryptographically secured?

DeDi.global entries are cryptographically secured through proof anchoring on the CORD blockchain—a Distributed Ledger Technology (DLT) that serves as a permanent, tamper-evident reserve of proofs.

### Overview

Every directory entry in DeDi.global is backed by a cryptographic proof anchored on-chain. This ensures that:

* **Authenticity** is provable—every entry can be traced to a verified publisher
* **Integrity** is guaranteed—any tampering is detectable
* **Provenance** is auditable—the complete history of changes is preserved

***

### Identity Foundation

When a user registers on DeDi.global, a cryptographic identity is established on the CORD chain.

#### Profile Creation

1. A **keypair** is generated using the **sr25519 algorithm** (a Schnorr signature scheme optimized for blockchain use)
2. Using this keypair, a **profile** is created on-chain with a unique `profile_id`
3. This `profile_id` appears as the `created_by` value across namespaces, registries, and records

#### Key Management

* Each `profile_id` is associated with an sr25519 keypair
* **Key rotation** is supported at the protocol level
* After rotation, the `profile_id` always resolves to the latest public key
* All signing and transaction fees are handled using the currently associated keypair

***

### DID Documents

For every `profile_id`, a **DID (Decentralized Identifier) document** is automatically generated. The URI follows the pattern:

```
did:web:did.cord.network:{id}
```

You can access the DID document for any `namespace_id` or `created_by` value at:

```
https://did.cord.network/{id}/did.json
```

***

### How Proof Anchoring Works

All registry and record proofs are submitted through a profile and signed using the corresponding keypair. From the blockchain's perspective, anchoring a proof is a **transaction**.

#### The Anchoring Process

1. **Data Serialization**: The entry data (registry or record) is structured into a blob
2. **Hashing**: The blob is serialized via `JSON.stringify()` and hashed using the **BLAKE2 H256** algorithm
3. **Signing**: The digest (hash) and blob form an unsigned transaction, which is then signed using the private key associated with the `profile_id`
4. **Submission**: The signed transaction is submitted to the chain for validation
5. **Inclusion**: If the signature and nonce are valid, the transaction is included in a block as a valid on-chain proof

#### Registry Proof Structure

When a registry is created or updated, the following data is used to generate the proof:

```javascript
{
  namespace_id,      // Parent namespace identifier
  registry_name,     // Human-readable name
  description,       // Registry description
  schema,            // Data schema definition
  tag,               // Classification tag
  version,           // Current version number
  version_count,     // Total version count
  state,             // Current state (active, revoked, etc.)
  genesis,           // Original creation reference
  created_by,        // Profile ID of creator
  meta,              // Additional metadata
  ttl                // Time-to-live configuration
}
```

#### Record Proof Structure

Record proofs follow a similar pattern, with the addition of the `details` field containing the actual record data:

```javascript
{
  namespace_id,
  registry_name,
  description,
  details,           // The actual record content
  tag,
  version,
  version_count,
  state,
  genesis,
  created_by,
  meta,
  ttl
}
```

***

### Chain Events and Identifiers

Once a transaction is successfully included in a block, an event is logged on the CORD chain.

#### Identifier Generation

* `registry_id` and `record_id` are **not** part of the original blob
* The chain **generates these identifiers automatically** upon successful anchoring
* This ensures identifier uniqueness and prevents collisions

#### Queryable Data

Using an identifier, you can query:

* Latest state of the entry
* All associated signatures
* Complete historical state changes

#### Version Tracking

Any update to a registry or record results in:

* A new blob and new digest
* The **same identifier** (preserving continuity)
* A new entry in the version history

***

### Verification

Anyone can verify a DeDi entry by:

1. **Retrieving the proof** from the CORD blockchain using the entry's identifier
2. **Reconstructing the digest** by hashing the entry data using BLAKE2 H256
3. **Comparing** the reconstructed digest with the on-chain digest
4. **Verifying the signature** against the public key associated with the `profile_id`

#### Blockchain Explorer

All proofs and transactions are publicly viewable at:

🔗 [apps.cord.network](https://apps.cord.network/)

Here you can inspect:

* Timestamps of all transactions
* Cryptographic signatures
* Transaction details and parameters
* Associated identifiers (`namespace_id`, `registry_id`, `record_id`)

***

### Summary

| Concept                 | Description                                                                         |
| ----------------------- | ----------------------------------------------------------------------------------- |
| **CORD**                | Distributed ledger anchoring all DeDi proofs                                        |
| **Profile**             | Unique on-chain identity created using sr25519 keys                                 |
| **DID Document**        | JSON document representing an on-chain identity                                     |
| **Proof Anchoring**     | Signed transactions storing content digests on-chain                                |
| **BLAKE2 H256**         | Cryptographic hash algorithm used for digest generation                             |
| **sr25519**             | Signature scheme for keypair generation and signing                                 |
| **Blockchain Explorer** | [apps.cord.network](https://apps.cord.network/)—view all proof and transaction data |
