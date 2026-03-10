# Research Papers Required for CipherFlow

This document details which research papers you need to read and how they apply to CipherFlow.

---

## 🎯 Critical Research Papers (Must Read)

### 1. NDAI Agreements ⭐⭐⭐
**AI agents enforcing NDAs via TEE**

- **Author**: Andrew Miller (Hackathon Organizer / Flashbots[X] / Associate Director, IC3)
- **Track**: TEE & AI-Enabled Applications
- **Priority**: CRITICAL - This is the foundation

#### Why This Paper Matters
This paper directly addresses your core use case: **AI processing with privacy guarantees**. It shows how to use TEEs to enforce NDAs with AI agents, which is exactly what CipherFlow needs for secure pitch deck evaluation.

#### Key Concepts to Extract
1. **TEE-based AI Execution**
   - How to run AI agents inside TEEs
   - Isolation guarantees
   - Attestation mechanisms

2. **NDA Enforcement Mechanisms**
   - How to ensure AI doesn't leak data
   - Contract enforcement patterns
   - Privacy guarantees

3. **AI Agent Architecture**
   - Secure AI pipeline design
   - Input/output controls
   - Verification patterns

#### How to Apply to CipherFlow
- Use TEE isolation for AI processing
- Implement similar privacy guarantees
- Adapt NDA enforcement for document processing
- Use attestation for proof generation

#### Reading Focus
- **Sections**: TEE architecture, AI agent design, NDA enforcement
- **Skip**: Specific NDA legal frameworks (focus on technical)
- **Take Notes**: Architecture patterns, security guarantees

---

### 2. Props ⭐⭐⭐
**Privacy-preserving ML data pipelines using zkTLS**

- **Author**: Dani Vilardell (PhD Student, Cornell University)
- **Track**: TEE & AI-Enabled Applications
- **Priority**: CRITICAL - Exact use case match

#### Why This Paper Matters
Props is about **privacy-preserving ML data pipelines** - this is exactly what CipherFlow does! It uses zkTLS for verifiable TLS connections and TEE-based processing.

#### Key Concepts to Extract
1. **zkTLS for Verifiable Connections**
   - Zero-knowledge proofs for TLS
   - Verifiable data transmission
   - Privacy-preserving connections

2. **ML Pipeline Architecture**
   - Secure data flow design
   - Pipeline isolation
   - End-to-end encryption

3. **TEE-based Processing**
   - Containerized ML execution
   - Secure computation patterns
   - Result verification

#### How to Apply to CipherFlow
- Use similar pipeline architecture
- Implement verifiable data transmission
- Adapt ML pipeline patterns for AI analysis
- Use zkTLS concepts for secure connections

#### Reading Focus
- **Sections**: Pipeline architecture, zkTLS implementation, TEE integration
- **Skip**: Specific ML model details (focus on infrastructure)
- **Take Notes**: Pipeline design, verification mechanisms

---

### 3. Conditional Recall ⭐⭐
**Brokered credential delegation via TEE**

- **Author**: Xyn Sun (Entrepreneur in Residence, Teleport/Flashbots[X])
- **Track**: TEE & AI-Enabled Applications
- **Priority**: HIGH - Key sharing mechanism

#### Why This Paper Matters
CipherFlow's **ShadowVault** concept needs secure key sharing. Conditional Recall shows how to delegate credentials via TEE without exposing secrets - perfect for our key management system.

#### Key Concepts to Extract
1. **Credential Delegation**
   - Secure credential brokering
   - TEE-based delegation
   - Conditional access patterns

2. **Key Management**
   - Secure key storage
   - Delegation mechanisms
   - Access control

3. **TEE Integration**
   - Secure key reconstruction
   - Isolated key operations
   - Attestation for keys

#### How to Apply to CipherFlow
- Implement ShadowVault using similar patterns
- Use TEE for key reconstruction
- Adapt delegation for threshold cryptography
- Secure key brokering architecture

#### Reading Focus
- **Sections**: Delegation architecture, TEE key operations, security model
- **Skip**: Specific credential formats (focus on patterns)
- **Take Notes**: Key management patterns, delegation mechanisms

---

### 4. Thetacrypt ⭐⭐
**Modular Rust library for threshold signatures, encryption, and verifiable randomness**

- **Author**: Mariarosaria Barbaraci (PhD Student, University of Bern)
- **Track**: Cryptographic Primitives & Identity
- **Priority**: HIGH - Threshold cryptography implementation

#### Why This Paper Matters
ShadowVault uses **threshold cryptography** to split keys. Thetacrypt provides a modular library for threshold operations - we can use it as a reference for our implementation.

#### Key Concepts to Extract
1. **Threshold Signatures**
   - Multi-party signature schemes
   - Key splitting mechanisms
   - Signature reconstruction

2. **Threshold Encryption**
   - Multi-party encryption
   - Key sharing patterns
   - Decryption protocols

3. **Verifiable Randomness**
   - Random number generation
   - Verification mechanisms
   - Security guarantees

#### How to Apply to CipherFlow
- Use threshold cryptography for ShadowVault
- Implement key splitting (3-of-3 threshold)
- Adapt signature schemes for proof generation
- Reference implementation patterns

#### Reading Focus
- **Sections**: Threshold schemes, key splitting, security proofs
- **Skip**: Rust-specific implementation (focus on algorithms)
- **Take Notes**: Threshold algorithms, security properties

---

### 5. Narrowing the Gap: TEE Threat Models & Deployment ⭐⭐
**What TEE attestation can and can't do today**

- **Author**: Filip Rezabek (PhD Student, Technical University of Munich)
- **Track**: Cryptographic Primitives & Identity
- **Priority**: ESSENTIAL - Realistic threat model

#### Why This Paper Matters
You need to understand **what TEEs can actually do** and their limitations. This paper provides realistic threat models and deployment considerations - critical for security design.

#### Key Concepts to Extract
1. **TEE Capabilities**
   - What TEEs can guarantee
   - Attestation capabilities
   - Isolation guarantees

2. **TEE Limitations**
   - What TEEs cannot do
   - Threat model boundaries
   - Attack vectors

3. **Deployment Considerations**
   - Real-world deployment challenges
   - Practical security considerations
   - Best practices

#### How to Apply to CipherFlow
- Design realistic security guarantees
- Understand threat model boundaries
- Implement appropriate mitigations
- Set correct expectations

#### Reading Focus
- **Sections**: Threat models, capabilities, limitations, deployment
- **Skip**: Specific hardware details (focus on concepts)
- **Take Notes**: Threat model, security boundaries, mitigations

---

## 📖 Secondary Research Papers (Recommended)

### 6. Cycles Protocol
**Privacy-preserving multilateral clearing and settlement via TEE**

- **Author**: Ethan Buchman (CEO Informal Systems / Co-founder Cosmos)
- **Track**: TEE & AI-Enabled Applications
- **Priority**: OPTIONAL - Attribution tracking

#### Why This Might Be Useful
Could be used for **attribution/contribution tracking** if you want to track who contributed what in collaborative scenarios. Not essential for MVP.

#### Key Concepts
- TEE-based clearing
- Graph processing for attribution
- Privacy-preserving settlement

---

### 7. Dstack Infrastructure
**Decentralized TEE cloud**

- **Track**: TEE & AI-Enabled Applications (Infrastructure)
- **Priority**: REFERENCE - Future deployment

#### Why This Might Be Useful
Understanding the **Dstack infrastructure layer** could be useful for future deployment. Not needed for MVP but good to understand the ecosystem.

---

## 📚 Reading Strategy

### Phase 1: Foundation (Start Here)
1. **NDAI Agreements** - Core concept
2. **Props** - Pipeline architecture
3. **TEE Threat Models** - Security foundation

**Time**: 4-6 hours
**Goal**: Understand core concepts and architecture

### Phase 2: Implementation Details
4. **Conditional Recall** - Key management
5. **Thetacrypt** - Threshold cryptography

**Time**: 3-4 hours
**Goal**: Understand implementation patterns

### Phase 3: Optional Enhancements
6. **Cycles Protocol** - Attribution (if time)
7. **Dstack** - Infrastructure (reference)

**Time**: 2-3 hours
**Goal**: Additional features and deployment

---

## 📝 Research Notes Template

For each paper, create notes with:

### Paper: [Name]
- **Core Innovation**: What problem does it solve?
- **Technical Approach**: How does it work?
- **Key Algorithms**: What algorithms/techniques?
- **Relevant Components**: What can we use?
- **Implementation Notes**: How to adapt?
- **Security Properties**: What guarantees?
- **Limitations**: What can't it do?
- **Code References**: Any code repositories?

---

## 🔍 Finding the Papers

### Where to Find Papers
1. **IC3 Website** - Check IC3 research publications
2. **arXiv** - Search by author names
3. **GitHub** - Some papers have code repositories
4. **Hackathon Resources** - Organizers may provide links

### Search Terms
- "NDAI Agreements" + "Andrew Miller"
- "Props" + "zkTLS" + "Dani Vilardell"
- "Conditional Recall" + "TEE" + "Xyn Sun"
- "Thetacrypt" + "threshold" + "Mariarosaria Barbaraci"
- "TEE Threat Models" + "Filip Rezabek"

### Code Repositories
Some papers mention code:
- **Props** - Check for GitHub repo
- **Conditional Recall** - Teleport project
- **Thetacrypt** - Rust library repository
- **Cycles Protocol** - Code mentioned

---

## ✅ Pre-Reading Checklist

Before reading, ensure you understand:
- [ ] Basic TEE concepts (what are TEEs?)
- [ ] Cryptography fundamentals (encryption, signatures)
- [ ] Zero-knowledge proofs basics
- [ ] Containerization (Docker basics)

### Quick TEE Primer
- **TEE**: Trusted Execution Environment
- **Isolation**: Code runs in hardware-isolated zone
- **Attestation**: Proof that code ran in TEE
- **Enclave**: Secure execution environment

---

## 🎯 Post-Reading Action Items

After reading each paper:
1. **Summarize** - Write 1-page summary
2. **Extract** - Identify relevant components
3. **Adapt** - Plan how to use in CipherFlow
4. **Document** - Add to implementation notes

---

## 📖 Additional Resources

### General TEE Resources
- Intel SGX documentation
- AWS Nitro Enclaves
- Confidential Computing Consortium

### Cryptography Resources
- Threshold cryptography tutorials
- Zero-knowledge proof basics
- Secure multi-party computation

### AI/ML Privacy Resources
- Privacy-preserving ML
- Federated learning
- Homomorphic encryption basics

---

## Questions to Answer While Reading

For each paper, answer:
1. **What problem does this solve?**
2. **How does it solve it?**
3. **What can we use directly?**
4. **What needs adaptation?**
5. **What are the security guarantees?**
6. **What are the limitations?**

---

## Next Steps

1. **Find the papers** (IC3 website, arXiv, GitHub)
2. **Read Phase 1 papers** (NDAI, Props, Threat Models)
3. **Take detailed notes** using the template
4. **Identify implementation patterns**
5. **Start building** with research insights

Good luck! 🚀

