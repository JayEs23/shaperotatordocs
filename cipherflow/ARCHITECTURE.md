# CipherFlow - Technical Architecture

Deep dive into the system architecture, components, and design decisions.

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Component Design](#component-design)
3. [Data Flow](#data-flow)
4. [Security Architecture](#security-architecture)
5. [API Design](#api-design)
6. [Database Schema](#database-schema)
7. [Deployment Architecture](#deployment-architecture)

---

## System Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      CipherFlow Platform                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Frontend Layer (Next.js)               │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │  Upload  │  │ Dashboard │  │  Judge   │        │    │
│  │  │   Page   │  │   Page    │  │  Page    │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            API Layer (Next.js API Routes)            │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │ /upload  │  │ /process │  │ /results │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Business Logic Layer                     │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │ Crypto   │  │  Parser  │  │    AI    │        │    │
│  │  │  Module  │  │  Module  │  │  Engine  │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Data Layer (PostgreSQL)                   │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │Documents │  │   Jobs    │  │ Results  │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │         Secure Worker Container (Docker)             │    │
│  │  ┌──────────────────────────────────────────────┐  │    │
│  │  │  TEE-Simulated Enclave                      │  │    │
│  │  │  • Decrypt Document                         │  │    │
│  │  │  • Extract Text                             │  │    │
│  │  │  • AI Analysis                              │  │    │
│  │  │  • Generate Proof                           │  │    │
│  │  └──────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
│                          │                                    │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            ShadowVault (Threshold Keys)              │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │    │
│  │  │ System   │  │   User   │  │ Enclave  │        │    │
│  │  │ Fragment │  │ Fragment │  │ Fragment │        │    │
│  │  └──────────┘  └──────────┘  └──────────┘        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Design

### 1. Frontend Layer

#### Technology Stack
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State**: React Query + Zustand
- **Forms**: React Hook Form + Zod

#### Key Components

**Upload Page** (`app/upload/page.tsx`)
- File upload interface
- Drag-and-drop support
- Progress tracking
- Metadata input

**Dashboard** (`app/dashboard/page.tsx`)
- Job list view
- Status indicators
- Result previews
- Download options

**Judge Dashboard** (`app/judge/page.tsx`)
- Analysis summaries
- Score displays
- Proof verification
- Comparison views

### 2. API Layer

#### Route Structure

```
/api
├── upload/
│   └── route.ts          # POST - Upload document
├── process/
│   └── route.ts          # POST - Trigger processing
├── results/
│   └── [id]/
│       └── route.ts      # GET - Get results
└── proof/
    └── [id]/
        └── route.ts      # GET - Verify proof
```

#### API Design Principles
- RESTful endpoints
- JSON request/response
- Error handling with status codes
- Authentication middleware
- Rate limiting

### 3. Business Logic Layer

#### Crypto Module (`lib/crypto/`)

**encryption.ts**
```typescript
- encryptDocument(file: Buffer): EncryptedData
- decryptDocument(encrypted: EncryptedData, key: Key): Buffer
- generateKey(): Key
- hashDocument(file: Buffer): string
```

**shadowvault.ts**
```typescript
- splitKey(key: Key): KeyFragment[]
- reconstructKey(fragments: KeyFragment[]): Key
- storeFragment(fragment: KeyFragment, location: Location)
- retrieveFragment(location: Location): KeyFragment
```

**signatures.ts**
```typescript
- signData(data: Buffer, key: Key): Signature
- verifySignature(data: Buffer, signature: Signature, key: Key): boolean
- generateProof(job: Job, result: Result): Proof
```

#### Parser Module (`lib/parser/`)

**pdf.ts**
```typescript
- extractText(file: Buffer): string
- extractMetadata(file: Buffer): Metadata
```

**docx.ts**
```typescript
- extractText(file: Buffer): string
- extractMetadata(file: Buffer): Metadata
```

**pptx.ts**
```typescript
- extractText(file: Buffer): string
- extractSlides(file: Buffer): Slide[]
```

#### AI Engine (`lib/ai/`)

**analyzer.ts**
```typescript
- analyzePitchDeck(text: string): Analysis
- generateSummary(text: string): Summary
- extractInsights(text: string): Insights
```

**prompts.ts**
```typescript
- getPitchDeckPrompt(): string
- getAnalysisPrompt(): string
- getScoringPrompt(): string
```

### 4. Secure Worker Container

#### Container Design

**Dockerfile Structure**
```dockerfile
FROM node:20-alpine

# Install dependencies
COPY package.json .
RUN npm ci --production

# Copy worker code
COPY workers/ ./workers/
COPY lib/ ./lib/

# Set up secure environment
ENV NODE_ENV=production
ENV SECURE_MODE=true

# Run worker
CMD ["node", "workers/enclave-worker.js"]
```

#### Worker Entry Point (`workers/enclave-worker.ts`)

```typescript
async function processJob(jobId: string) {
  // 1. Retrieve encrypted document
  const encryptedDoc = await getEncryptedDocument(jobId);
  
  // 2. Reconstruct decryption key (inside enclave)
  const key = await reconstructKey(jobId);
  
  // 3. Decrypt document
  const document = decryptDocument(encryptedDoc, key);
  
  // 4. Extract text
  const text = await extractText(document);
  
  // 5. Run AI analysis
  const analysis = await analyzeWithAI(text);
  
  // 6. Generate proof
  const proof = await generateProof(jobId, document, analysis);
  
  // 7. Sign result
  const signedResult = await signResult(analysis, proof);
  
  // 8. Return result
  return signedResult;
}
```

#### Security Measures
- Network restrictions (no outbound except API)
- File system restrictions
- Process isolation
- Resource limits
- Attestation generation

### 5. ShadowVault System

#### Architecture

```
┌─────────────────────────────────────────┐
│         ShadowVault Key System          │
├─────────────────────────────────────────┤
│                                         │
│  Original Key: [K]                      │
│         │                                │
│         ▼                                │
│  ┌──────────┐  ┌──────────┐  ┌────────┐│
│  │ Fragment │  │ Fragment │  │Fragment││
│  │    1     │  │    2     │  │   3    ││
│  └──────────┘  └──────────┘  └────────┘│
│       │            │            │        │
│       ▼            ▼            ▼        │
│  ┌──────────┐  ┌──────────┐  ┌────────┐│
│  │ System   │  │   User   │  │Enclave ││
│  │ Storage  │  │  Session │  │Memory  ││
│  └──────────┘  └──────────┘  └────────┘│
│                                         │
│  Reconstruction: Only inside enclave    │
└─────────────────────────────────────────┘
```

#### Key Splitting Algorithm
- **Scheme**: 3-of-3 threshold
- **Method**: Shamir Secret Sharing (or simpler XOR split)
- **Storage**: Distributed fragments
- **Reconstruction**: Only in secure enclave

---

## Data Flow

### Upload Flow

```
1. User selects file
   ↓
2. Frontend validates file
   ↓
3. POST /api/upload
   ↓
4. Generate document hash
   ↓
5. Encrypt document
   ↓
6. Store encrypted file
   ↓
7. Create database record
   ↓
8. Return job ID
```

### Processing Flow

```
1. POST /api/process { jobId }
   ↓
2. Create processing job
   ↓
3. Queue job for worker
   ↓
4. Start secure container
   ↓
5. Container retrieves encrypted doc
   ↓
6. Reconstruct key (inside enclave)
   ↓
7. Decrypt document
   ↓
8. Extract text content
   ↓
9. Call AI API
   ↓
10. Generate analysis
   ↓
11. Create execution proof
   ↓
12. Sign result
   ↓
13. Store result + proof
   ↓
14. Update job status
   ↓
15. Return result
```

### Result Retrieval Flow

```
1. GET /api/results/[id]
   ↓
2. Retrieve result from DB
   ↓
3. Verify proof signature
   ↓
4. Return result + proof
   ↓
5. Frontend displays analysis
```

---

## Security Architecture

### Encryption Layers

```
┌─────────────────────────────────────┐
│  Layer 1: Transport (HTTPS/TLS)     │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Layer 2: Application (AES-256)      │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Layer 3: Storage (Encrypted DB)    │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│  Layer 4: Enclave Isolation         │
└─────────────────────────────────────┘
```

### Threat Mitigation

| Threat | Layer | Mitigation |
|--------|-------|-----------|
| Network Eavesdropping | Transport | TLS encryption |
| Application Attack | Application | AES-256 encryption |
| Database Breach | Storage | Encrypted storage |
| Insider Access | Enclave | Container isolation |
| Key Compromise | ShadowVault | Threshold cryptography |

---

## API Design

### Request/Response Patterns

**Upload Endpoint**
```typescript
POST /api/upload
Content-Type: multipart/form-data

Request:
{
  file: File,
  metadata?: {
    title?: string
  }
}

Response:
{
  jobId: string,
  documentHash: string,
  timestamp: string
}
```

**Process Endpoint**
```typescript
POST /api/process
Content-Type: application/json

Request:
{
  jobId: string
}

Response:
{
  status: "queued" | "processing",
  jobId: string,
  estimatedTime?: number
}
```

**Results Endpoint**
```typescript
GET /api/results/[id]

Response:
{
  jobId: string,
  status: "completed",
  analysis: {
    startupName: string,
    problem: string,
    solution: string,
    market: string,
    strengths: string[],
    weaknesses: string[],
    score: number
  },
  proof: {
    documentHash: string,
    timestamp: string,
    signature: string,
    containerAttestation: string
  }
}
```

---

## Database Schema

See `DATABASE_SCHEMA.md` for complete schema.

### Key Tables

- **documents** - Encrypted document storage
- **jobs** - Processing job tracking
- **results** - Analysis results
- **proofs** - Execution proofs
- **key_fragments** - ShadowVault key fragments

---

## Deployment Architecture

### Development

```
┌─────────────┐
│   Next.js   │
│  (Port 3000)│
└─────────────┘
      │
      ▼
┌─────────────┐
│ PostgreSQL  │
│ (Port 5432) │
└─────────────┘
      │
      ▼
┌─────────────┐
│   Docker    │
│  Container  │
└─────────────┘
```

### Production

```
┌─────────────┐
│   Vercel    │
│  (Next.js)  │
└─────────────┘
      │
      ▼
┌─────────────┐
│   Supabase  │
│ (PostgreSQL)│
└─────────────┘
      │
      ▼
┌─────────────┐
│   Railway   │
│  (Workers)  │
└─────────────┘
```

---

## Design Decisions

### Why Next.js Only?
- **Simplicity**: Single codebase, single language
- **Speed**: Faster development, less context switching
- **Deployment**: Easy Vercel deployment
- **Hackathon**: Perfect for MVP scope

### Why Docker for Worker?
- **Isolation**: Simulates TEE isolation
- **Portability**: Runs anywhere
- **Security**: Container restrictions
- **Future**: Easy migration to real TEEs

### Why Threshold Cryptography?
- **Security**: No single point of failure
- **Trust**: Minimizes trust requirements
- **Research**: Aligns with hackathon research
- **Practical**: Solves real key management problem

---

## Next Steps

1. Review this architecture
2. Read research papers
3. Start implementation
4. Iterate based on learnings

