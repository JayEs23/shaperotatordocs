# Velarc – TEE Threat Model (Narrowing the Gap) Notes

**Paper focus**: What TEEs *actually* guarantee vs how they are deployed in real clouds, and how to design realistic threat models and deployment strategies.

---

## 1. Core Idea (In Plain Language)

- TEEs promise:
  - **Confidentiality**: Host/OS/cloud cannot read enclave memory.
  - **Integrity**: Host cannot tamper with enclave code without detection.
  - **Attestation**: Remote parties can verify which code is running.

- In practice:
  - Cloud deployments often **weaken** or misconfigure these guarantees:
    - Complex stacks (firmware, hypervisor, BIOS, drivers) add attack surface.
    - Attestation flows are hard to use correctly.
    - Side channels and misconfig can leak data.

- For Velarc:
  - We must **not oversell** security.
  - We should design with a **realistic, conservative threat model**, even though our prototype simulates TEEs.

---

## 2. Threat Model for Velarc (Inspired by the Paper)

### 2.1 What We Treat as Adversarial

- **Cloud provider / host OS / hypervisor**:
  - May read disks, network traffic, logs.
  - May attempt to run modified images or downgrade code.

- **Platform operator**:
  - Has access to infra, metrics, logs, DB.
  - Should still be unable to read raw pitch decks.

### 2.2 What We Trust (In the Future, with Real TEEs)

- CPU / TEE hardware implementation (with known limitations).
- TEE vendor attestation service.
- Enclave code we ship (our own code, measured and reviewed).

### 2.3 For the Prototype (TEE Simulation)

- We **cannot claim hardware TEE guarantees**, but:
  - We can **structure code** as if we were targeting real TEEs.
  - We can **simulate attestation** and isolation:
    - Dedicated worker image.
    - No raw documents outside worker.
    - Clear boundaries and checks.

---

## 3. Deployment Lessons to Apply to Velarc

### 3.1 Minimal Trusted Computing Base (TCB)

**Paper insight**: The more code and components inside the TCB, the harder it is to reason about security.

**Velarc adaptation**:
- Keep the **worker container** as small and focused as possible:
  - Only:
    - Decrypt
    - Parse
    - Call AI
    - Apply policy
    - Generate proof
  - Avoid:
    - Extra web frameworks
    - Unnecessary libraries and tools.

Design decision:
- Separate:
  - Next.js app (frontend/API)
  - Worker image (minimal Node/crypto/AI client)

---

### 3.2 Attestation Chain Awareness

**Paper insight**: Real-world TEE deployments involve many layers (firmware, bootloaders, VMM, etc.), and attestation should reflect this chain, not just “enclave code hash”.

**Velarc adaptation (prototype)**:
- Our “attestation” object should include:
  - Worker image digest
  - Base image identifier
  - Policy version
  - Build/commit hash

In a real TEE deployment, this would be extended with:
  - TEE report
  - Vendor quote
  - Cloud platform evidence.

---

### 3.3 Side-Channel and Metadata Risks

**Paper insight**: Even if memory is protected, **side channels** (timing, memory patterns, I/O volume) can leak information.

**Velarc adaptation**:
- For prototype:
  - We **acknowledge** these risks and document them.
  - We avoid obvious leaks in:
    - Logs (no input-dependent logging of content).
    - Error messages (no echoing of user data).
    - Result sizes (avoid directly proportional size to input).

Documentation requirement:
- In `SECURITY.md`, clearly state:
  - We simulate TEE properties.
  - Side channels are not fully mitigated in the prototype.

---

## 4. Concrete Security Design Decisions for Velarc

1. **No Raw Documents Outside Worker**  
   - API and DB never store plaintext.
   - Only encrypted blobs + hashes.

2. **Strict Worker API**  
   - Worker receives:
     - Job ID
     - Location of encrypted blob
     - Key material (or way to fetch it) after attestation.
   - Worker returns:
     - `AnalysisResult`
     - `ProcessingProof`
   - Nothing else (no raw content).

3. **Simulated Attestation Requirements**  
   - ShadowVault must see:
     - Known `workerImageDigest`
     - Known `policyVersion`
     - Known `codeVersion` (commit hash)
   - If anything mismatches: **deny key release**.

4. **Logging Policy**  
   - No logging of:
     - Raw document bytes
     - Large excerpts from AI output
   - Only:
     - Job IDs
     - Status
     - Hashes and digests

5. **Key Management Boundaries**  
   - Key shares live outside worker until attestation accepted.
   - Key reconstruction only happens inside worker memory.

---

## 5. Implementation Checklist for Velarc (Threat-Model-Driven)

- [ ] **Worker Design**
  - Create a minimal worker runtime with only required dependencies.
  - Ensure no HTTP server in worker unless absolutely needed.

- [ ] **Attestation Object**
  - Define `WorkerAttestation` type:
    - `workerImageDigest`
    - `policyVersion`
    - `codeVersion`
    - `timestamp`
  - Implement:
    - `generateWorkerAttestation()` in worker.
    - `verifyWorkerAttestation()` in ShadowVault.

- [ ] **Key Release Logic**
  - In ShadowVault:
    - Reject key releases if attestation fails or is stale.
    - Log all key release attempts.

- [ ] **Logging / Telemetry**
  - Central logging that:
    - Never logs document content.
    - Tags logs by `jobId` and `documentHash`.

- [ ] **Security Documentation**
  - In `SECURITY.md`, add:
    - Our **assumed threat model** (what we protect against).
    - Our **out-of-scope risks** (side channels, compromised client, etc.).
    - A clear note that this is a **prototype TEE simulation**, not a full production TEE deployment.

These notes translate the “Narrowing the Gap” paper into concrete architecture and coding decisions that keep Velarc honest about what it can and cannot guarantee, while still moving toward a real TEE-based design in the future.


