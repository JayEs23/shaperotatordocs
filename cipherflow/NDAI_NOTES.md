# Velarc – NDAI Agreements Research Notes

**Paper focus**: AI agents that can access sensitive data under an NDA, but are technically prevented from leaking that data outside agreed channels, using TEEs and attestation.

---

## 1. Core Idea (In Plain Language)

- **What NDAI solves**:  
  Organizations want to let an AI model *see* sensitive inputs (e.g. pitch decks) but **don’t trust**:
  - The cloud provider
  - The model host
  - The humans operating the system  
  NDAI uses TEEs + attestation to force AI execution to follow a hard-coded “NDA policy”.

- **Key concept**:  
  - The model runs **inside a TEE**.  
  - The TEE produces an **attestation** proving:
    - Which code is running (measurement / hash)
    - Which model / policy version
  - Clients only send encrypted data to TEEs with a valid attestation.
  - Outputs are strictly controlled (no raw data, only allowed summaries / scores / proofs).

---

## 2. Threat Model – What We Should Assume for Velarc

- **TEE is trusted for**:
  - Code isolation (no one outside can read process memory)
  - Code integrity (loaded code matches the attested measurement)
  - Attestation (remote parties can verify which code is running)

- **We do NOT fully trust**:
  - Cloud provider (can see disk/network but not enclave memory)
  - Host OS / hypervisor
  - Human operators of the platform

- **For Velarc prototype** (TEE-simulated):
  - We **simulate** this with:
    - A dedicated Docker worker image whose hash we control
    - A “fake attestation” object that includes:
      - Docker image digest
      - Git commit hash
      - Build timestamp
  - We must design so that **in the future** we can plug in *real* TEE attestation (e.g. SEV-SNP/TDX).

---

## 3. Key Mechanisms to Mirror in Velarc

### 3.1 Policy-Bound Execution

**NDAI idea**: The AI agent’s behavior is constrained by a *policy* baked into enclave code (e.g. “only emit classifications/summaries, never raw text”).  

**Velarc adaptation**:
- We implement an internal **`PolicyEngine`** in the worker:
  - Allowed outputs:
    - Structured summary
    - Scores
    - Limited-length text snippets (if ever needed, with caps)
  - Forbidden outputs:
    - Full document text
    - Arbitrary free-form “echo of input”
- We treat the **policy version** as part of the “measurement” for attestation.

**Concrete design**:
- `lib/policy/policyEngine.ts` (or similar) that:
  - Takes the raw AI result
  - Filters / truncates / transforms into allowed schema
  - Logs which policy version was applied

---

### 3.2 Attestation-Gated Decryption

**NDAI idea**: Client only releases decryption keys if TEE’s attestation proves the right code/policy is loaded.

**Velarc adaptation (ShadowVault)**:
- ShadowVault holds **threshold-split decryption keys** (or grants access to them).
- To reconstruct a key inside the worker:
  1. Worker boots and creates a **self-attestation object** (simulated).
  2. ShadowVault verifies:
     - Image digest matches an approved list.
     - Code version (Git commit) is approved.
     - Policy version is approved.
  3. Only then does ShadowVault release its key share or grant a one-time decrypt token.

**Concrete design hooks**:
- `lib/shadowvault/attestation.ts`
  - `generateWorkerAttestation()` – collect image hash, commit, config.
  - `verifyWorkerAttestation(attestation)` – run policy checks.
- `lib/shadowvault/keys.ts`
  - `requestKeyShare(attestation)` – only succeed if attestation passes checks.

---

### 3.3 Output Channel Control

**NDAI idea**: Even if model “knows” the secret, outputs are restricted to safe channels / formats.

**Velarc adaptation**:
- Worker must **only** send data back via:
  - Database write of **structured results**
  - Cryptographic proof object
- No raw document, no arbitrary logs containing secrets.

**Concrete design points**:
- Logging:
  - No logs of raw text or large excerpts.
  - Only log hashes, job IDs, status codes, and high-level metrics.
- Result schema:
  - Strict TypeScript types for `AnalysisResult`:
    - No `rawText`, no `fullDocument`, no `largeExcerpt`.

---

## 4. How NDAI Shapes Velarc’s Architecture

### 4.1 Required Components (Mapping from NDAI to Velarc)

- **TEE / Enclave** → Velarc Secure Worker Container
  - Docker image with:
    - Decrypt → Parse → Call AI → PolicyFilter → Sign + Store
  - Future: Replace with actual TEE runtime.

- **Attestation Service** → Velarc Attestation Module
  - For prototype:
    - Sign a JSON object locally (e.g. HMAC or dev key).
    - Store image digest and policy version.
  - Future:
    - Integrate with cloud TEE attestation (Azure, GCP, etc.).

- **Policy Engine** → Velarc Policy Module
  - Encodes what judges are allowed to see.

- **NDA Contract** → Velarc “Analysis Terms”
  - For hackathon: documented assumptions:
    - Judges see only structured outputs.
    - Platform operators cannot read raw decks.

---

## 5. Implementation Checklist for Velarc (NDAI-Driven)

- [ ] **Define a strict `AnalysisResult` schema** with:
  - Startup name, problem, solution, market
  - Strengths/weaknesses
  - Score
  - No raw document fields.
- [ ] **Implement `PolicyEngine`**:
  - Input: model raw output (JSON / text)
  - Output: `AnalysisResult` + policy metadata
  - Enforce max lengths, allowed keys only.
- [ ] **Implement simulated attestation**:
  - `generateWorkerAttestation()` gathers:
    - Docker image digest (configured env)
    - Git commit hash
    - Policy version
  - `verifyWorkerAttestation()` in ShadowVault before releasing key shares.
- [ ] **Ensure all decryption happens only inside the worker**:
  - API layer never handles plaintext documents.
- [ ] **Audit all output channels**:
  - No route returns raw documents.
  - Logs + DB schema cannot accidentally store secrets in clear.

These are the concrete hooks where NDAI’s ideas become Velarc behavior.


