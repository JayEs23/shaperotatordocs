# Velarc - Database Schema

Complete database schema design using Prisma ORM.

---

## Schema Overview

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

---

## Core Tables

### 1. Users Table

```prisma
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  role          UserRole  @default(PARTICIPANT)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  documents     Document[]
  jobs          Job[]
  
  @@index([email])
}
```

**User Roles**:
- `PARTICIPANT` - Hackathon participant
- `JUDGE` - Hackathon judge
- `ADMIN` - System administrator

---

### 2. Documents Table

```prisma
model Document {
  id                String    @id @default(cuid())
  userId            String
  user              User      @relation(fields: [userId], references: [id])
  
  // File information
  originalFilename  String
  mimeType          String
  fileSize          Int       // bytes
  encryptedFilePath String    // Path to encrypted file storage
  
  // Security
  documentHash      String    @unique // SHA-256 hash of original file
  encryptionKeyHash String    // Hash of encryption key (for verification)
  
  // Metadata
  title             String?
  description       String?
  
  // Timestamps
  uploadedAt        DateTime  @default(now())
  deletedAt         DateTime?
  
  // Relations
  jobs              Job[]
  
  @@index([userId])
  @@index([documentHash])
  @@index([uploadedAt])
}
```

**Fields**:
- `encryptedFilePath`: Path to encrypted file (S3, local storage, etc.)
- `documentHash`: Cryptographic hash for integrity verification
- `encryptionKeyHash`: Hash of encryption key (stored for verification)

---

### 3. Jobs Table

```prisma
model Job {
  id            String      @id @default(cuid())
  documentId    String
  document      Document    @relation(fields: [documentId], references: [id])
  userId        String
  user          User        @relation(fields: [userId], references: [id])
  
  // Status tracking
  status        JobStatus   @default(QUEUED)
  priority      Int         @default(0)
  
  // Processing
  startedAt     DateTime?
  completedAt   DateTime?
  failedAt      DateTime?
  errorMessage  String?
  
  // Worker information
  workerId      String?     // Container/worker identifier
  containerId   String?     // Docker container ID
  
  // Timestamps
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
  
  // Relations
  result        Result?
  proof         Proof?
  
  @@index([userId])
  @@index([documentId])
  @@index([status])
  @@index([createdAt])
}
```

**Job Statuses**:
- `QUEUED` - Waiting to be processed
- `PROCESSING` - Currently being processed
- `COMPLETED` - Successfully completed
- `FAILED` - Processing failed
- `CANCELLED` - Cancelled by user

---

### 4. Results Table

```prisma
model Result {
  id            String    @id @default(cuid())
  jobId         String    @unique
  job           Job       @relation(fields: [jobId], references: [id], onDelete: Cascade)
  
  // Analysis results
  startupName   String?
  problem       String?
  solution      String?
  market        String?
  traction      String?
  
  // Structured data
  strengths     String[]  // Array of strengths
  weaknesses    String[]  // Array of weaknesses
  
  // Scoring
  score         Float?    // 1-10 scale
  investmentReadiness String? // "High" | "Medium" | "Low"
  
  // Raw AI response (for debugging)
  rawResponse   String?   @db.Text
  
  // Timestamps
  createdAt     DateTime  @default(now())
  
  @@index([jobId])
}
```

**Analysis Fields**:
- Structured extraction from AI analysis
- Score and investment readiness
- Raw response stored for debugging

---

### 5. Proofs Table

```prisma
model Proof {
  id                    String    @id @default(cuid())
  jobId                 String    @unique
  job                   Job       @relation(fields: [jobId], references: [id], onDelete: Cascade)
  
  // Document verification
  documentHash          String    // Must match Document.documentHash
  documentHashVerified  Boolean   @default(false)
  
  // Execution proof
  executionTimestamp    DateTime
  containerAttestation  String    // Container/enclave attestation
  workerSignature       String    // Signature from worker
  
  // Result verification
  resultHash            String    // Hash of analysis result
  resultSignature       String    // Signature of result
  
  // Key information
  keyFragmentHash       String?   // Hash of key fragment used
  
  // Verification status
  verified              Boolean   @default(false)
  verifiedAt            DateTime?
  
  // Timestamps
  createdAt             DateTime  @default(now())
  
  @@index([jobId])
  @@index([documentHash])
  @@index([verified])
}
```

**Proof Components**:
- Document hash verification
- Container attestation
- Worker signature
- Result signature
- Verification status

---

### 6. Key Fragments Table (ShadowVault)

```prisma
model KeyFragment {
  id            String          @id @default(cuid())
  documentId    String
  document      Document        @relation(fields: [documentId], references: [id], onDelete: Cascade)
  
  // Fragment information
  fragmentIndex Int             // 1, 2, or 3 (for 3-of-3 threshold)
  fragmentHash  String          // Hash of fragment (for verification)
  encryptedFragment String      // Encrypted fragment data
  
  // Storage location
  storageLocation FragmentLocation
  storagePath    String          // Path/identifier in storage location
  
  // Security
  encryptedAt    DateTime        @default(now())
  expiresAt      DateTime?       // Optional expiration
  
  // Timestamps
  createdAt      DateTime        @default(now())
  
  @@unique([documentId, fragmentIndex])
  @@index([documentId])
  @@index([storageLocation])
}
```

**Fragment Locations**:
- `SYSTEM` - System storage (database)
- `USER` - User session storage
- `ENCLAVE` - Enclave memory (temporary)

---

### 7. Processing Queue Table (Optional)

```prisma
model ProcessingQueue {
  id            String      @id @default(cuid())
  jobId         String      @unique
  job           Job         @relation(fields: [jobId], references: [id], onDelete: Cascade)
  
  // Queue management
  priority      Int         @default(0)
  retryCount    Int         @default(0)
  maxRetries    Int         @default(3)
  
  // Scheduling
  scheduledAt   DateTime    @default(now())
  startedAt     DateTime?
  
  // Status
  status        QueueStatus @default(PENDING)
  
  // Timestamps
  createdAt     DateTime    @default(now())
  updatedAt     DateTime    @updatedAt
  
  @@index([status, priority, scheduledAt])
  @@index([jobId])
}
```

**Queue Statuses**:
- `PENDING` - Waiting in queue
- `PROCESSING` - Currently processing
- `COMPLETED` - Successfully processed
- `FAILED` - Failed processing
- `CANCELLED` - Cancelled

---

## Enums

```prisma
enum UserRole {
  PARTICIPANT
  JUDGE
  ADMIN
}

enum JobStatus {
  QUEUED
  PROCESSING
  COMPLETED
  FAILED
  CANCELLED
}

enum FragmentLocation {
  SYSTEM
  USER
  ENCLAVE
}

enum QueueStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
  CANCELLED
}
```

---

## Relationships Diagram

```
User
 ├── documents (1:N)
 └── jobs (1:N)

Document
 ├── user (N:1)
 └── jobs (1:N)

Job
 ├── document (N:1)
 ├── user (N:1)
 ├── result (1:1)
 ├── proof (1:1)
 └── queueEntry (1:1)

Result
 └── job (1:1)

Proof
 └── job (1:1)

KeyFragment
 └── document (N:1)

ProcessingQueue
 └── job (1:1)
```

---

## Indexes

### Performance Indexes

```prisma
// User lookups
@@index([email])
@@index([role])

// Document lookups
@@index([userId])
@@index([documentHash])
@@index([uploadedAt])

// Job queries
@@index([userId])
@@index([documentId])
@@index([status])
@@index([createdAt])
@@index([status, createdAt]) // Composite for status queries

// Proof verification
@@index([documentHash])
@@index([verified])

// Queue processing
@@index([status, priority, scheduledAt])
```

---

## Migrations

### Initial Migration

```bash
# Generate migration
npx prisma migrate dev --name init

# Apply migration
npx prisma migrate deploy
```

### Migration Strategy

1. **Development**: Use `prisma migrate dev`
2. **Production**: Use `prisma migrate deploy`
3. **Schema Changes**: Always create new migrations

---

## Seed Data (Optional)

```prisma
// prisma/seed.ts

async function main() {
  // Create test users
  const judge = await prisma.user.create({
    data: {
      email: "judge@hackathon.com",
      name: "Judge User",
      role: "JUDGE",
    },
  });
  
  const participant = await prisma.user.create({
    data: {
      email: "participant@hackathon.com",
      name: "Participant User",
      role: "PARTICIPANT",
    },
  });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

---

## Prisma Client Usage

### Example Queries

```typescript
// Create document
const document = await prisma.document.create({
  data: {
    userId: user.id,
    originalFilename: file.name,
    mimeType: file.type,
    fileSize: file.size,
    encryptedFilePath: "/storage/encrypted/file.pdf",
    documentHash: hash,
  },
});

// Create job
const job = await prisma.job.create({
  data: {
    documentId: document.id,
    userId: user.id,
    status: "QUEUED",
  },
});

// Get job with relations
const job = await prisma.job.findUnique({
  where: { id: jobId },
  include: {
    document: true,
    result: true,
    proof: true,
  },
});

// Update job status
await prisma.job.update({
  where: { id: jobId },
  data: {
    status: "COMPLETED",
    completedAt: new Date(),
  },
});
```

---

## Database Considerations

### Storage
- **Encrypted Files**: Store in object storage (S3, etc.), not database
- **Database**: Store metadata and references only
- **Key Fragments**: Encrypted fragments can be stored in DB

### Performance
- **Indexes**: Critical for job queries and status updates
- **Partitioning**: Consider partitioning jobs table by date (future)
- **Archiving**: Archive old jobs/results (future)

### Security
- **Encryption at Rest**: Enable PostgreSQL encryption
- **Connection Security**: Use SSL/TLS for connections
- **Access Control**: Use database roles and permissions

---

## Next Steps

1. **Set up Prisma**: `npx prisma init`
2. **Create schema**: Copy schema to `prisma/schema.prisma`
3. **Generate client**: `npx prisma generate`
4. **Run migrations**: `npx prisma migrate dev`
5. **Start building**: Use Prisma Client in code

