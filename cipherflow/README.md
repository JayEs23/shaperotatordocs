# Velarc - Complete Documentation

**Confidential AI Execution Platform for Secure Document Analysis**

---

## 📚 Documentation Index

1. [Product Overview](#product-overview)
2. [Research Papers Required](#research-papers-required)
3. [Technical Architecture](#technical-architecture)
4. [Implementation Guide](#implementation-guide)
5. [Security Design](#security-design)
6. [API Reference](#api-reference)
7. [Deployment Guide](#deployment-guide)
8. [Development Roadmap](#development-roadmap)
9. [Demo Video Guide](#demo-video-guide)

**Additional Documents:**
- `QUICK_START.md` - Your starting point and roadmap
- `RESEARCH_PAPERS.md` - Detailed research paper analysis
- `ARCHITECTURE.md` - Complete technical architecture
- `DATABASE_SCHEMA.md` - Database design
- `SECURITY.md` - Security design and threat model
- `IMPLEMENTATION_GUIDE.md` - Step-by-step build guide
- `DEMO_VIDEO.md` - Remotion-based demo video guide

---

## Product Overview

### What is Velarc?

Velarc is a **confidential AI execution platform** that enables sensitive documents to be analyzed by AI **without exposing the raw data** to the platform, operators, or third-party services.

### Core Value Proposition

- 🔒 **Zero-trust AI processing** - Documents encrypted before processing
- 🛡️ **TEE-simulated isolation** - Secure containerized execution environment
- ✅ **Cryptographic proofs** - Verifiable computation integrity
- 🔑 **Threshold key management** - ShadowVault distributed key system

### Use Cases

1. **Hackathon Pitch Deck Evaluation** (Primary Demo)
   - Judges receive AI summaries without seeing raw pitch decks
   - Protects IP during evaluation process

2. **Confidential Research Analysis**
   - Process proprietary research documents securely
   - Enable AI insights without data exposure

3. **Enterprise Document Processing**
   - Financial reports, legal documents, strategic plans
   - AI analysis with privacy guarantees

---

## Research Papers Required

### 🎯 Primary Research Papers (Must Read)

#### 1. **NDAI Agreements** ⭐ CRITICAL
- **Author**: Andrew Miller (Hackathon Organizer / Flashbots[X] / IC3)
- **Why**: Directly addresses AI agents enforcing NDAs via TEE
- **Relevance**: Core concept - AI processing with privacy guarantees
- **Key Concepts**:
  - TEE-based AI agent execution
  - NDA enforcement mechanisms
  - Privacy-preserving AI workflows
- **How to Use**: Foundation for secure AI execution model

#### 2. **Props** ⭐ CRITICAL
- **Author**: Dani Vilardell (PhD Student, Cornell University)
- **Why**: Privacy-preserving ML data pipelines using zkTLS
- **Relevance**: Exact use case - secure data pipelines for ML/AI
- **Key Concepts**:
  - zkTLS for verifiable TLS connections
  - Privacy-preserving ML pipelines
  - TEE-based data processing
- **How to Use**: Architecture pattern for encrypted data pipelines

#### 3. **Conditional Recall** ⭐ HIGH PRIORITY
- **Author**: Xyn Sun (Teleport/Flashbots[X])
- **Why**: Brokered credential delegation via TEE
- **Relevance**: ShadowVault key sharing mechanism
- **Key Concepts**:
  - TEE-based credential delegation
  - Secure key brokering
  - Conditional access patterns
- **How to Use**: Key sharing and delegation architecture

#### 4. **Thetacrypt** ⭐ HIGH PRIORITY
- **Author**: Mariarosaria Barbaraci (PhD Student, University of Bern)
- **Why**: Modular threshold cryptography library
- **Relevance**: ShadowVault threshold key splitting
- **Key Concepts**:
  - Threshold signatures
  - Threshold encryption
  - Verifiable randomness
- **How to Use**: Implementation reference for key splitting

#### 5. **Narrowing the Gap: TEE Threat Models & Deployment** ⭐ ESSENTIAL
- **Author**: Filip Rezabek (PhD Student, Technical University of Munich)
- **Why**: Understanding TEE limitations and capabilities
- **Relevance**: Realistic threat model for TEE simulation
- **Key Concepts**:
  - TEE attestation capabilities
  - Threat model boundaries
  - Deployment considerations
- **How to Use**: Security design and threat modeling

### 📖 Secondary Research Papers (Recommended)

#### 6. **Cycles Protocol**
- **Author**: Ethan Buchman (Informal Systems / Cosmos)
- **Why**: Privacy-preserving multilateral clearing
- **Relevance**: Could be used for attribution/contribution tracking
- **Key Concepts**: TEE-based clearing and settlement

#### 7. **Dstack Infrastructure**
- **Why**: Decentralized TEE cloud infrastructure
- **Relevance**: Future deployment target
- **Note**: Understanding the infrastructure layer

### 🔍 Research Reading Strategy

1. **Start with**: NDAI Agreements + Props (core concepts)
2. **Then**: Conditional Recall + Thetacrypt (key management)
3. **Finally**: TEE Threat Models (security foundation)
4. **Reference**: Cycles Protocol (optional enhancements)

### 📝 Research Notes Template

For each paper, document:
- **Core Innovation**: What problem does it solve?
- **Technical Approach**: How does it work?
- **Relevant Components**: What can we use?
- **Implementation Notes**: How to adapt for Velarc?

---

## Technical Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────┐
│                      Velarc Platform                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐         ┌──────────────┐             │
│  │   Frontend   │────────▶│   API Layer  │             │
│  │  (Next.js)   │         │  (Next.js)   │             │
│  └──────────────┘         └──────────────┘             │
│         │                        │                      │
│         │                        ▼                      │
│         │              ┌─────────────────┐             │
│         │              │  Encrypted       │             │
│         │              │  Storage         │             │
│         │              │  (PostgreSQL)    │             │
│         │              └─────────────────┘             │
│         │                        │                      │
│         │                        ▼                      │
│         │              ┌─────────────────┐             │
│         │              │  Job Queue      │             │
│         │              └─────────────────┘             │
│         │                        │                      │
│         └────────────────────────┼──────────────────────┘
│                                  │
│                                  ▼
│                    ┌─────────────────────────┐
│                    │  Secure Worker Container │
│                    │  (TEE Simulation)       │
│                    ├─────────────────────────┤
│                    │ • Decrypt Document      │
│                    │ • Extract Text           │
│                    │ • AI Analysis            │
│                    │ • Generate Proof         │
│                    └─────────────────────────┘
│                                  │
│                                  ▼
│                    ┌─────────────────────────┐
│                    │  ShadowVault            │
│                    │  (Threshold Keys)       │
│                    └─────────────────────────┘
│                                  │
│                                  ▼
│                    ┌─────────────────────────┐
│                    │  Results + Proofs       │
│                    └─────────────────────────┘
└─────────────────────────────────────────────────────────┘
```

### Technology Stack

#### Frontend + Backend
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **State Management**: React Query / Zustand

#### Database
- **Primary**: PostgreSQL
- **ORM**: Prisma
- **Migrations**: Prisma Migrate

#### AI Processing
- **Provider**: OpenAI API (MVP)
- **Future**: Local models in enclave
- **SDK**: OpenAI Node.js SDK

#### Document Processing
- **PDF**: `pdf-parse` or `pdfjs-dist`
- **DOCX**: `mammoth`
- **PPTX**: `officegen` or custom parser

#### Cryptography
- **Core**: Node.js `crypto` module
- **Advanced**: `tweetnacl` for signatures
- **Key Management**: Custom threshold implementation

#### Containerization
- **Runtime**: Docker
- **Orchestration**: Docker Compose (dev)
- **Future**: Kubernetes (production)

### Data Flow

```
1. User Uploads Document
   ↓
2. Frontend → API Route
   ↓
3. Generate Document Hash
   ↓
4. Encrypt Document (AES-256)
   ↓
5. Store Encrypted File + Metadata
   ↓
6. Create Processing Job
   ↓
7. Queue Job for Worker
   ↓
8. Secure Worker Container Starts
   ↓
9. Decrypt Document (inside enclave)
   ↓
10. Extract Text Content
   ↓
11. Call AI API (with encrypted context)
   ↓
12. Generate Structured Analysis
   ↓
13. Create Execution Proof
   ↓
14. Sign Results
   ↓
15. Store Results + Proof
   ↓
16. Return to Frontend
```

---

## Implementation Guide

### Phase 1: Project Setup

#### 1.1 Initialize Next.js Project

```bash
npx create-next-app@latest velarc --typescript --tailwind --app
cd velarc
```

#### 1.2 Install Dependencies

```bash
# Core
npm install prisma @prisma/client
npm install zod react-hook-form
npm install @tanstack/react-query zustand

# AI & Processing
npm install openai
npm install pdf-parse mammoth

# Crypto
npm install tweetnacl
npm install @noble/hashes  # Additional crypto utilities

# Docker
npm install dockerode  # Docker API client

# Utilities
npm install date-fns
npm install clsx tailwind-merge
```

#### 1.3 Database Setup

```bash
# Initialize Prisma
npx prisma init

# Configure PostgreSQL connection
# Update .env with DATABASE_URL
```

#### 1.4 Docker Setup

Create `docker/enclave.Dockerfile` for secure worker.

### Phase 2: Database Schema

See `DATABASE_SCHEMA.md` for complete schema design.

### Phase 3: Core Components

1. **Encryption Module** (`lib/crypto/`)
2. **Document Parser** (`lib/parser/`)
3. **AI Engine** (`lib/ai/`)
4. **Proof Generator** (`lib/proof/`)
5. **ShadowVault** (`lib/shadowvault/`)

### Phase 4: API Routes

1. `/api/upload` - Document upload
2. `/api/process` - Trigger processing
3. `/api/results/[id]` - Get results
4. `/api/proof/[id]` - Verify proof

### Phase 5: Frontend Pages

1. `/upload` - Upload interface
2. `/dashboard` - User dashboard
3. `/judge` - Judge dashboard
4. `/results/[id]` - Results viewer

### Phase 6: Secure Worker

1. Docker container setup
2. Worker entry point
3. Decryption logic
4. AI processing
5. Proof generation

---

## Security Design

### Threat Model

| Threat | Mitigation |
|--------|-----------|
| Data Leakage | AES-256 encryption, encrypted storage |
| Insider Access | TEE simulation, container isolation |
| Tampering | Cryptographic signatures, proof verification |
| Result Manipulation | Signed results, hash verification |
| Key Compromise | Threshold cryptography, key splitting |

### Security Principles

1. **Encrypt Everything**: Documents encrypted before storage
2. **Isolate Processing**: Secure container with restricted access
3. **Verify Everything**: Cryptographic proofs for all operations
4. **Minimize Trust**: No single party has full access
5. **Audit Trail**: All operations logged and signed

### ShadowVault Design

- **Key Splitting**: 3-of-3 threshold (system, user, enclave)
- **Reconstruction**: Only inside secure enclave
- **Storage**: Fragments stored separately
- **Rotation**: Periodic key rotation support

---

## API Reference

### POST `/api/upload`

Upload a document for processing.

**Request**:
```typescript
{
  file: File,
  metadata?: {
    title?: string,
    description?: string
  }
}
```

**Response**:
```typescript
{
  jobId: string,
  documentHash: string,
  timestamp: string
}
```

### POST `/api/process`

Trigger processing for a document.

**Request**:
```typescript
{
  jobId: string
}
```

**Response**:
```typescript
{
  status: "queued" | "processing" | "completed" | "failed",
  jobId: string
}
```

### GET `/api/results/[id]`

Get analysis results.

**Response**:
```typescript
{
  jobId: string,
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

## Deployment Guide

### Development

```bash
# Start database
docker-compose up -d postgres

# Run migrations
npx prisma migrate dev

# Start dev server
npm run dev
```

### Production

1. Deploy Next.js to Vercel/Railway
2. Set up PostgreSQL (Supabase/Neon)
3. Configure Docker registry
4. Set environment variables
5. Deploy secure worker containers

---

## Development Roadmap

### Week 1: Foundation
- [ ] Project setup
- [ ] Database schema
- [ ] Basic encryption
- [ ] Document upload

### Week 2: Core Features
- [ ] Secure worker container
- [ ] AI integration
- [ ] Proof generation
- [ ] Basic UI

### Week 3: Polish
- [ ] Judge dashboard
- [ ] Proof verification
- [ ] UI/UX improvements
- [ ] Testing

### Week 4: Demo Prep
- [ ] End-to-end testing
- [ ] Documentation
- [ ] Demo script
- [ ] Presentation

---

## Next Steps

1. **Read Research Papers** (see RESEARCH_PAPERS.md)
2. **Set Up Development Environment**
3. **Implement Core Components**
4. **Build Incrementally**
5. **Test Thoroughly**

---

## Demo Video

Create a compelling demo video using **Remotion** (React-based video creation).

See `DEMO_VIDEO.md` for:
- Complete video script and storyboard
- Remotion setup and components
- Screen recording tips
- Editing workflow
- Production checklist

**Key Scenes:**
1. Problem statement (0:00-0:30)
2. Solution introduction (0:30-0:45)
3. Live demo - Upload (0:45-1:15)
4. Live demo - Processing (1:15-1:45)
5. Live demo - Results (1:45-2:15)
6. Technical deep dive (2:15-2:40)
7. Call to action (2:40-3:00)

---

## Questions?

Refer to:
- `QUICK_START.md` - Your starting point and roadmap
- `RESEARCH_PAPERS.md` - Detailed research paper analysis
- `ARCHITECTURE.md` - Deep dive into system design
- `IMPLEMENTATION_GUIDE.md` - Step-by-step build guide
- `DEMO_VIDEO.md` - Remotion-based demo video guide
- `SECURITY.md` - Security design and threat model

