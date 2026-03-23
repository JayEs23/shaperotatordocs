# Velarc – PROPS Research Notes

**Paper focus**: Privacy-preserving ML / AI data pipelines using zkTLS (zero-knowledge TLS) to prove that data was processed correctly over encrypted channels, without revealing the data itself.

---

## 1. Core Idea (In Plain Language)

- **What PROPS solves**:  
  - Today, when you send data to an ML/AI service over TLS, you must *trust*:
    - The TLS endpoint
    - The service operator
  - PROPS uses **zkTLS** to create a **cryptographic proof** that:
    - A specific program ran
    - On specific (encrypted) inputs
    - Following a specific protocol
  - The verifier learns the **result is correct**, but not the raw data.

- **Why this matters for Velarc**:
  - Velarc wants to process pitch decks **without exposing raw content** to infrastructure.
  - We need a **verifiable pipeline**: encrypt → process → output, with evidence that the pipeline followed the rules.

---

## 2. Threat Model – How It Guides Velarc

- Assume:
  - Network endpoints and infrastructure **might be malicious or misconfigured**.
  - We want a judge or participant to **verify**:
    - The document was encrypted before leaving their machine.
    - It was only decrypted inside an approved worker context.
    - The AI results came from that approved context.

- For Velarc prototype:
  - We **won’t implement full zkTLS**, but we can:
    - Model the same **pipeline structure**.
    - Emit a **simplified “pipeline proof object”** that captures:
      - Input hash
      - Encryption step metadata
      - Worker image and policy version
      - AI model identifier
      - Timestamps and signatures

---

## 3. Key Pipeline Concepts to Mirror in Velarc

### 3.1 End-to-End Encrypted Pipeline

**PROPS idea**: Data is encrypted at the client, stays encrypted in transit/storage, and is only handled in cleartext inside controlled execution.

**Velarc pipeline**:
1. **Client / Frontend**:
   - Compute `documentHash = SHA-256(fileBytes)`.
   - Encrypt file (`AES-256-GCM`) with a fresh symmetric key.
   - Upload **only** the encrypted blob + metadata.
2. **Backend / Storage**:
   - Store encrypted blob + `documentHash` + job metadata.
3. **Worker**:
   - After attestation check, reconstruct or retrieve key.
   - Decrypt inside worker.
   - Run parsing + AI analysis + policy filter.

**Design decision**:
- Never store or transport raw bytes outside the worker container.

---

### 3.2 Verifiable Pipeline Steps (Proof Object)

**PROPS idea**: zk proofs bind the steps of the pipeline into a verifiable statement.

**Velarc adaptation** (simplified, no heavy ZK for prototype):

- Define a **`ProcessingProof`** object:

```markdown
ProcessingProof {
  documentHash: string        // SHA-256 of original file
  encryptedBlobHash: string   // SHA-256 of encrypted file
  workerImageDigest: string   // Docker image hash
  policyVersion: string       // PolicyEngine version
  modelId: string             // e.g. gpt-4o-2024-xx
  startedAt: string
  finishedAt: string
  signature: string           // Signing key held by Velarc/worker
}
```

- This object is:
  - Stored with results.
  - Returned via `/api/results/[id]`.
  - Verifiable by re-hashing data + checking signature.

**Later**:
- This structure can be upgraded to include **real zkTLS/TEE attestation evidence** while preserving the same API shape.

---

### 3.3 Separation of Roles

**PROPS emphasizes**:
- Clear separation between:
  - Data owners (participants)
  - Compute providers (TEE/worker)
  - Verifiers (judges)

**Velarc mapping**:
- **Participant / Team**:
  - Uploads encrypted pitch deck.
  - Can locally verify `documentHash`.
- **Velarc Worker**:
  - Decrypts, analyzes, produces results + `ProcessingProof`.
- **Judge**:
  - Reads only:
    - Structured analysis
    - Proof object
  - Optionally verifies:
    - Proof signature
    - That the worker image/policy is an approved one.

---

## 4. How PROPS Changes Velarc’s Design

### 4.1 Required Data Fields

Velarc must track the following for each job:

- `documentHash`: hash of raw file (client-side or server-side before deletion)
- `encryptedBlobHash`: hash of encrypted file at rest
- `encryptionAlgorithm`: e.g. `AES-256-GCM`
- `workerImageDigest`
- `policyVersion`
- `modelId`
- `processingTimestamps`
- `signature`

These become the building blocks of the **proof story**.

---

### 4.2 API Changes (Inspired by PROPS)

- **Upload API** (`POST /api/upload`):
  - Request additions:
    - Optional `clientDocumentHash`
  - Backend:
    - Always computes its own `serverDocumentHash` and stores both.
    - Computes `encryptedBlobHash`.

- **Results API** (`GET /api/results/[id]`):
  - Response additions:
    - `proof: ProcessingProof`

This makes the pipeline **transparent and checkable**, even without full zkTLS.

---

## 5. Implementation Checklist for Velarc (PROPS-Driven)

- [ ] **Encryption Flow**
  - Implement `encryptDocument(fileBytes)` → `{ encryptedBytes, key, iv }`.
  - Compute `documentHash` and `encryptedBlobHash`.
  - Store hashes + algorithm in DB.

- [ ] **ProcessingProof Object**
  - Define a TypeScript type:
    - `ProcessingProof` (as above).
  - Implement `generateProcessingProof(context)` in worker:
    - Fills in hashes, digests, timestamps, modelId.
    - Signs with a worker key.
  - Store in DB and expose via API.

- [ ] **Signing and Verification**
  - Worker has a signing keypair (dev key for now).
  - Backend / client has `workerPublicKey`.
  - Implement `verifyProcessingProof(proof, publicKey)` for:
    - Internal checks
    - Optional client-side verification.

- [ ] **UI Hooks**
  - Show a **“Verified Pipeline”** badge on results:
    - Green if proof verifies.
    - Grey if proof missing/invalid.

These steps apply PROPS’ core idea—**verifiable encrypted pipelines**—to Velarc in a way that fits a hackathon prototype, while leaving room for upgrading to full zkTLS later.


