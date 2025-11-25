# DeDi in Verifiable Credential Workflows

DeDi serves as a **directory and trust layer** for verifiable credential (VC) ecosystems—complementing DIDs and DID Documents by providing discoverable, auditable registries of issuers, keys, schemas, and credential status.

***

### The Verifier's Problem

When a verifier receives a credential, they need answers to five questions:

1. Is this DID valid and current?
2. Which public key should verify the signature?
3. Was this key valid when the credential was issued?
4. Has this credential been revoked?
5. Is this issuer trusted in my trust network?

DID Documents answer questions 1–2 for a single identifier. DeDi addresses all five across entire ecosystems.

***

### What DeDi Provides

#### 1. Unified Key Discovery

**Problem:** Public keys are scattered across email headers (S/MIME), key servers (PGP), DNS (DANE), certificate transparency logs, and proprietary directories. No single place to ask: "What is this entity's current public key?"

**DeDi solution:** Organizations publish their keys to a claimed namespace on DeDi. Verifiers query one standardized API regardless of the underlying DID method.

#### 2. Auditable Key History

**Problem:** DID Documents show current state only. When keys rotate, there's no record of which key was valid at what time—breaking verification of older credentials.

**DeDi solution:** Every key change is a versioned record with cryptographic provenance anchored on-chain. Verifiers can retrieve the key that was valid at any point in time. Subscribers can receive alerts when keys change, ensuring updates propagate reliably.

#### 3. Revocation and Status Lists

**Problem:** Each issuer maintains bespoke revocation registries with different formats, endpoints, and update frequencies. Verifiers can't efficiently check status across issuers.

**DeDi solution:** Issuers publish status lists (revoked credentials, suspended issuers, approved institutions) as standardized, machine-readable registries. Updates propagate immediately and are tamper-evident via on-chain anchoring. Verifiers can subscribe to status changes for real-time alerts.

#### 4. Trusted Issuer Directories

**Problem:** Cryptographic validity doesn't answer "Should I trust this issuer?" Without shared trust infrastructure, verifiers make independent assessments for every new issuer.

**DeDi solution:** Authoritative bodies (regulators, accreditors, industry associations) publish directories of recognized issuers. Verifiers configure which namespaces they trust, then automatically accept issuers listed under those namespaces.

#### 5. VC Schema Registry

**Problem:** Credential schemas are often hosted on arbitrary URLs that may change or disappear. Schema versioning is inconsistent, making it hard to verify credentials against the correct schema version.

**DeDi solution:** Issuers publish VC schemas as registry records with full version history. Every schema change is tracked with provenance (who changed it, when, what changed). Verifiers can retrieve the exact schema version referenced by a credential.

***

### How It Works

#### Issuer Setup

1. Register a namespace on DeDi (e.g., `acme-corp`)
2. Publish registries for:
   * **Public keys** — current and historical
   * **DIDs** — with DID Document endpoints
   * **VC schemas** — with version tracking
   * **Status lists** — revocations, suspensions
3. Each publish operation is signed and anchored on-chain

#### Credential Issuance

1. Issuer signs the VC with their current key
2. VC contains:
   * Issuer DID
   * Schema reference (optionally pointing to DeDi)
   * Status list reference (optionally pointing to DeDi)

#### Verification

```
┌─────────────────────────────────────────────────────────┐
│  Verifier receives VC with issuer DID                   │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Query DeDi:                                            │
│  • Is issuer in a trusted namespace?                    │
│  • What key was valid at issuance time?                 │
│  • Is credential/issuer on any status list?             │
│  • What schema version applies?                         │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Resolve DID Document, cross-check key with DeDi        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  Verify signature → Accept or Reject                    │
└─────────────────────────────────────────────────────────┘
```

DeDi complements DID Documents: **DID Document** = cryptographic material for one identity; **DeDi** = discovery, trust context, and status across many identities.

***

### Summary

DeDi doesn't replace DIDs—it makes them **usable at scale** by answering the questions that DID Documents alone cannot:

* **Where** do I find trusted issuers?
* **What** key was valid when this credential was issued?
* **Is** this credential still valid?
* **Who** recognizes this issuer?
* **Which** schema version should I validate against?

All answers are machine-readable, API-accessible, and cryptographically anchored.
