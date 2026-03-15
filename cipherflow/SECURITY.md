# CipherFlow - Security Design

Complete security architecture, threat model, and mitigation strategies.

---

## Table of Contents

1. [Security Principles](#security-principles)
2. [Threat Model](#threat-model)
3. [Security Architecture](#security-architecture)
4. [Cryptographic Design](#cryptographic-design)
5. [Container Security](#container-security)
6. [Key Management](#key-management)
7. [Proof System](#proof-system)
8. [Security Checklist](#security-checklist)

---

## Security Principles

### 1. Defense in Depth
Multiple layers of security:
- Transport encryption (TLS)
- Application encryption (AES-256)
- Storage encryption (encrypted database)
- Runtime isolation (container)

### 2. Zero Trust
- Never trust, always verify
- Encrypt everything
- Verify all operations
- Assume breach mentality

### 3. Least Privilege
- Minimal access requirements
- Principle of least privilege
- Isolated execution environments
- Restricted network access

### 4. Cryptographic Proofs
- Verify all operations
- Sign all results
- Prove execution integrity
- Audit trail

---

## Threat Model

### Threat Categories

#### 1. Data Exposure Threats

**T1: Network Eavesdropping**
- **Risk**: High
- **Description**: Attacker intercepts data in transit
- **Mitigation**: TLS encryption for all connections
- **Status**: Standard HTTPS/TLS

**T2: Database Breach**
- **Risk**: High
- **Description**: Attacker gains access to database
- **Mitigation**: Encrypted storage, no plaintext data
- **Status**: AES-256 encrypted storage

**T3: Storage Compromise**
- **Risk**: Medium
- **Description**: Attacker accesses file storage
- **Mitigation**: Encrypted files, separate key management
- **Status**: Encrypted file storage

#### 2. Processing Threats

**T4: Insider Access**
- **Risk**: High
- **Description**: System administrator accesses data
- **Mitigation**: Container isolation, encrypted processing
- **Status**: Secure container isolation

**T5: Worker Compromise**
- **Risk**: Medium
- **Description**: Worker container compromised
- **Mitigation**: Isolated execution, restricted access
- **Status**: Docker container restrictions

**T6: AI Provider Leakage**
- **Risk**: Medium
- **Description**: AI provider logs/leaks data
- **Mitigation**: Encrypted context, minimal data exposure
- **Status**: Encrypted prompts, future: local models

#### 3. Key Management Threats

**T7: Key Compromise**
- **Risk**: Critical
- **Description**: Encryption key exposed
- **Mitigation**: Threshold cryptography, key splitting
- **Status**: ShadowVault 3-of-3 threshold

**T8: Key Reconstruction Attack**
- **Risk**: Medium
- **Description**: Attacker reconstructs key outside enclave
- **Mitigation**: Key reconstruction only in enclave
- **Status**: Enclave-only reconstruction

#### 4. Integrity Threats

**T9: Result Tampering**
- **Risk**: Medium
- **Description**: Attacker modifies analysis results
- **Mitigation**: Cryptographic signatures, proof verification
- **Status**: Signed results, proof system

**T10: Document Substitution**
- **Risk**: Low
- **Description**: Attacker substitutes different document
- **Mitigation**: Document hash verification
- **Status**: Hash verification in proof

---

## Security Architecture

### Multi-Layer Encryption

```
┌─────────────────────────────────────┐
│ Layer 1: Transport Security         │
│ • TLS 1.3                           │
│ • Certificate pinning               │
│ • HSTS                              │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│ Layer 2: Application Encryption     │
│ • AES-256-GCM                        │
│ • Authenticated encryption           │
│ • Key derivation (PBKDF2/Argon2)    │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│ Layer 3: Storage Encryption          │
│ • Encrypted database                 │
│ • Encrypted file storage             │
│ • Encrypted key fragments            │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│ Layer 4: Runtime Isolation           │
│ • Container isolation                │
│ • Network restrictions               │
│ • File system restrictions           │
└─────────────────────────────────────┘
```

---

## Cryptographic Design

### Encryption Scheme

**Algorithm**: AES-256-GCM
- **Key Size**: 256 bits
- **Mode**: GCM (authenticated encryption)
- **IV**: Random 96-bit IV per encryption
- **Tag**: 128-bit authentication tag

**Key Derivation**:
```typescript
// Key derivation function
const key = await crypto.subtle.deriveKey(
  {
    name: "PBKDF2",
    salt: randomSalt,
    iterations: 100000,
    hash: "SHA-256",
  },
  baseKeyMaterial,
  { name: "AES-GCM", length: 256 },
  false,
  ["encrypt", "decrypt"]
);
```

### Hashing

**Document Hash**: SHA-256
- Used for integrity verification
- Stored in database
- Included in proof

**Key Fragment Hash**: SHA-256
- Hash of key fragments
- Used for verification
- Prevents tampering

### Signatures

**Algorithm**: Ed25519 (via tweetnacl)
- **Key Size**: 256 bits
- **Signature Size**: 512 bits
- **Properties**: Fast, secure, compact

**Usage**:
- Worker signatures
- Result signatures
- Proof signatures

---

## Container Security

### Docker Security Configuration

**Network Restrictions**:
```dockerfile
# Only allow outbound to API endpoint
--network=isolated
--dns=8.8.8.8
```

**File System Restrictions**:
```dockerfile
# Read-only root filesystem
--read-only
--tmpfs /tmp
```

**Resource Limits**:
```dockerfile
# CPU and memory limits
--cpus="1.0"
--memory="2g"
```

**Capabilities**:
```dockerfile
# Drop all capabilities
--cap-drop=ALL
--security-opt=no-new-privileges
```

### Container Isolation

**Process Isolation**:
- Separate process namespace
- No host process access
- Isolated PID namespace

**Network Isolation**:
- Restricted outbound connections
- Only API endpoint access
- No inbound connections

**File System Isolation**:
- Read-only filesystem
- Temporary storage only
- No host filesystem access

---

## Key Management

### ShadowVault Architecture

**Key Splitting**:
- **Scheme**: 3-of-3 threshold
- **Method**: XOR-based splitting (simple) or Shamir Secret Sharing (advanced)
- **Fragments**: 3 fragments stored separately

**Fragment Storage**:
1. **System Fragment**: Encrypted in database
2. **User Fragment**: User session storage (encrypted)
3. **Enclave Fragment**: Temporary in enclave memory

**Key Reconstruction**:
- Only inside secure enclave
- Requires all 3 fragments
- Never reconstructed outside enclave
- Fragments destroyed after use

### Key Lifecycle

```
1. Key Generation
   ↓
2. Key Splitting (3 fragments)
   ↓
3. Fragment Encryption
   ↓
4. Fragment Storage (distributed)
   ↓
5. Fragment Retrieval (for processing)
   ↓
6. Key Reconstruction (in enclave)
   ↓
7. Key Usage (decryption)
   ↓
8. Fragment Destruction
```

### Key Rotation (Future)

- Periodic key rotation
- Re-encryption of documents
- Fragment rotation
- Migration strategy

---

## Proof System

### Proof Components

**Document Hash**:
- SHA-256 hash of original document
- Proves document integrity
- Prevents substitution

**Execution Timestamp**:
- Precise execution time
- Proves when processing occurred
- Prevents replay attacks

**Container Attestation**:
- Container/enclave identifier
- Proves execution environment
- Prevents execution outside enclave

**Worker Signature**:
- Ed25519 signature from worker
- Proves worker executed job
- Prevents result tampering

**Result Signature**:
- Ed25519 signature of result
- Proves result integrity
- Prevents modification

### Proof Verification

```typescript
async function verifyProof(proof: Proof, document: Document, result: Result) {
  // 1. Verify document hash
  const documentHash = await hashDocument(document);
  if (proof.documentHash !== documentHash) {
    return { valid: false, reason: "Document hash mismatch" };
  }
  
  // 2. Verify worker signature
  const workerPublicKey = await getWorkerPublicKey();
  const signatureValid = verifySignature(
    proof.workerSignature,
    result,
    workerPublicKey
  );
  if (!signatureValid) {
    return { valid: false, reason: "Invalid worker signature" };
  }
  
  // 3. Verify result signature
  const resultHash = await hashResult(result);
  const resultSignatureValid = verifySignature(
    proof.resultSignature,
    resultHash,
    workerPublicKey
  );
  if (!resultSignatureValid) {
    return { valid: false, reason: "Invalid result signature" };
  }
  
  // 4. Verify container attestation
  const attestationValid = await verifyAttestation(proof.containerAttestation);
  if (!attestationValid) {
    return { valid: false, reason: "Invalid container attestation" };
  }
  
  return { valid: true };
}
```

---

## Security Checklist

### Development

- [ ] All dependencies scanned for vulnerabilities
- [ ] No hardcoded secrets or keys
- [ ] Environment variables for all secrets
- [ ] Input validation on all endpoints
- [ ] Output sanitization
- [ ] Error handling doesn't leak information
- [ ] Logging doesn't include sensitive data

### Encryption

- [ ] AES-256-GCM for document encryption
- [ ] TLS 1.3 for all connections
- [ ] Encrypted database storage
- [ ] Encrypted file storage
- [ ] Key fragments encrypted separately

### Container Security

- [ ] Read-only filesystem
- [ ] Network restrictions
- [ ] Resource limits
- [ ] Capability dropping
- [ ] No privileged mode
- [ ] Security scanning

### Key Management

- [ ] Threshold key splitting (3-of-3)
- [ ] Key reconstruction only in enclave
- [ ] Fragments stored separately
- [ ] Key rotation capability
- [ ] Secure key generation

### Proof System

- [ ] Document hash verification
- [ ] Worker signature verification
- [ ] Result signature verification
- [ ] Container attestation
- [ ] Timestamp verification

### Testing

- [ ] Security testing
- [ ] Penetration testing
- [ ] Vulnerability scanning
- [ ] Code review
- [ ] Threat modeling review

---

## Security Best Practices

### Code Security

1. **Input Validation**: Validate all inputs
2. **Output Encoding**: Encode all outputs
3. **Error Handling**: Don't leak information
4. **Logging**: No sensitive data in logs
5. **Dependencies**: Keep dependencies updated

### Operational Security

1. **Secrets Management**: Use secret management service
2. **Access Control**: Principle of least privilege
3. **Monitoring**: Monitor for anomalies
4. **Incident Response**: Have a plan
5. **Backup Security**: Encrypt backups

### Compliance Considerations

1. **Data Privacy**: GDPR considerations
2. **Data Retention**: Retention policies
3. **Audit Trail**: Complete audit logs
4. **Access Logs**: Who accessed what
5. **Data Deletion**: Secure deletion

---

## Future Security Enhancements

### Phase 2

- [ ] Real TEE hardware (Intel SGX / AWS Nitro)
- [ ] Hardware attestation
- [ ] Enhanced key management
- [ ] Multi-party computation

### Phase 3

- [ ] Zero-knowledge proofs
- [ ] Homomorphic encryption
- [ ] Decentralized execution
- [ ] Advanced threat detection

---

## Security Resources

### References

- OWASP Top 10
- NIST Cybersecurity Framework
- CWE Top 25
- TEE Security Best Practices

### Tools

- Dependency scanning: `npm audit`
- Container scanning: Trivy, Clair
- Code analysis: ESLint security plugins
- Penetration testing: OWASP ZAP

---

## Questions?

Refer to:
- Architecture documentation
- API security guidelines
- Deployment security checklist

