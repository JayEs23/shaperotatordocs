# Velarc - Implementation Guide

Step-by-step guide to building Velarc from scratch.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Project Setup](#project-setup)
3. [Phase 1: Foundation](#phase-1-foundation)
4. [Phase 2: Core Features](#phase-2-core-features)
5. [Phase 3: Secure Worker](#phase-3-secure-worker)
6. [Phase 4: Frontend](#phase-4-frontend)
7. [Phase 5: Integration](#phase-5-integration)
8. [Phase 6: Testing](#phase-6-testing)
9. [Phase 7: Deployment](#phase-7-deployment)

---

## Prerequisites

### Required Software

- **Node.js**: v20 or higher
- **npm**: v9 or higher
- **PostgreSQL**: v14 or higher
- **Docker**: v20 or higher
- **Git**: Latest version

### Required Accounts

- **OpenAI API**: API key for AI processing
- **Database**: PostgreSQL instance (local or cloud)
- **Storage**: File storage (local or S3)

### Required Knowledge

- TypeScript/JavaScript
- Next.js basics
- Docker basics
- PostgreSQL basics
- Cryptography basics

---

## Project Setup

### Step 1: Initialize Next.js Project

```bash
# Create Next.js project
npx create-next-app@latest velarc --typescript --tailwind --app --no-src-dir

# Navigate to project
cd velarc

# Initialize git
git init
git add .
git commit -m "Initial commit"
```

### Step 2: Install Dependencies

```bash
# Core dependencies
npm install prisma @prisma/client
npm install zod react-hook-form @hookform/resolvers
npm install @tanstack/react-query zustand
npm install date-fns clsx tailwind-merge

# AI & Processing
npm install openai
npm install pdf-parse mammoth

# Cryptography
npm install tweetnacl
npm install @noble/hashes

# Docker
npm install dockerode
npm install --save-dev @types/dockerode

# Utilities
npm install dotenv
npm install multer @types/multer
```

### Step 3: Environment Setup

Create `.env.local`:

```env
# Database
DATABASE_URL="postgresql://user:password@localhost:5432/velarc"

# OpenAI
OPENAI_API_KEY="your-api-key-here"

# Application
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NODE_ENV="development"

# Storage
STORAGE_PATH="./storage"
STORAGE_ENCRYPTED_PATH="./storage/encrypted"

# Docker
DOCKER_SOCKET_PATH="/var/run/docker.sock"
```

### Step 4: Database Setup

```bash
# Initialize Prisma
npx prisma init

# Copy schema from DATABASE_SCHEMA.md
# Edit prisma/schema.prisma

# Generate Prisma Client
npx prisma generate

# Run migrations
npx prisma migrate dev --name init
```

---

## Phase 1: Foundation

### Step 1.1: Project Structure

Create folder structure:

```
velarc/
├── app/
│   ├── (auth)/
│   ├── upload/
│   ├── dashboard/
│   └── judge/
├── api/
│   ├── upload/
│   ├── process/
│   └── results/
├── lib/
│   ├── crypto/
│   ├── parser/
│   ├── ai/
│   ├── proof/
│   ├── shadowvault/
│   └── db/
├── workers/
│   └── enclave-worker.ts
├── docker/
│   └── enclave.Dockerfile
├── prisma/
│   └── schema.prisma
└── public/
```

### Step 1.2: Database Schema

Copy schema from `DATABASE_SCHEMA.md` to `prisma/schema.prisma`.

### Step 1.3: Prisma Client

Create `lib/db/prisma.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

### Step 1.4: Basic Configuration

Create `lib/config.ts`:

```typescript
export const config = {
  app: {
    url: process.env.NEXT_PUBLIC_APP_URL || 'http://localhost:3000',
  },
  openai: {
    apiKey: process.env.OPENAI_API_KEY!,
  },
  storage: {
    path: process.env.STORAGE_PATH || './storage',
    encryptedPath: process.env.STORAGE_ENCRYPTED_PATH || './storage/encrypted',
  },
  docker: {
    socketPath: process.env.DOCKER_SOCKET_PATH || '/var/run/docker.sock',
  },
};
```

---

## Phase 2: Core Features

### Step 2.1: Encryption Module

Create `lib/crypto/encryption.ts`:

```typescript
import crypto from 'crypto';

export interface EncryptedData {
  encrypted: Buffer;
  iv: Buffer;
  tag: Buffer;
}

export async function encryptDocument(
  data: Buffer,
  key: Buffer
): Promise<EncryptedData> {
  const iv = crypto.randomBytes(12); // 96-bit IV for GCM
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  
  const encrypted = Buffer.concat([
    cipher.update(data),
    cipher.final(),
  ]);
  
  const tag = cipher.getAuthTag();
  
  return { encrypted, iv, tag };
}

export async function decryptDocument(
  encryptedData: EncryptedData,
  key: Buffer
): Promise<Buffer> {
  const decipher = crypto.createDecipheriv(
    'aes-256-gcm',
    key,
    encryptedData.iv
  );
  
  decipher.setAuthTag(encryptedData.tag);
  
  return Buffer.concat([
    decipher.update(encryptedData.encrypted),
    decipher.final(),
  ]);
}

export function generateKey(): Buffer {
  return crypto.randomBytes(32); // 256 bits
}

export function hashDocument(data: Buffer): string {
  return crypto.createHash('sha256').update(data).digest('hex');
}
```

### Step 2.2: Document Parser

Create `lib/parser/pdf.ts`:

```typescript
import pdf from 'pdf-parse';

export async function extractTextFromPDF(file: Buffer): Promise<string> {
  const data = await pdf(file);
  return data.text;
}

export async function extractMetadataFromPDF(file: Buffer) {
  const data = await pdf(file);
  return {
    pages: data.numpages,
    info: data.info,
  };
}
```

Create `lib/parser/docx.ts`:

```typescript
import mammoth from 'mammoth';

export async function extractTextFromDOCX(file: Buffer): Promise<string> {
  const result = await mammoth.extractRawText({ buffer: file });
  return result.value;
}
```

### Step 2.3: AI Engine

Create `lib/ai/analyzer.ts`:

```typescript
import OpenAI from 'openai';
import { config } from '../config';

const openai = new OpenAI({
  apiKey: config.openai.apiKey,
});

export interface PitchDeckAnalysis {
  startupName: string;
  problem: string;
  solution: string;
  market: string;
  traction: string;
  strengths: string[];
  weaknesses: string[];
  score: number;
  investmentReadiness: 'High' | 'Medium' | 'Low';
}

export async function analyzePitchDeck(
  text: string
): Promise<PitchDeckAnalysis> {
  const prompt = `Analyze this startup pitch deck and extract the following information in JSON format:
{
  "startupName": "string",
  "problem": "string",
  "solution": "string",
  "market": "string",
  "traction": "string",
  "strengths": ["string"],
  "weaknesses": ["string"],
  "score": number (1-10),
  "investmentReadiness": "High" | "Medium" | "Low"
}

Pitch deck text:
${text}`;

  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      {
        role: 'system',
        content: 'You are an expert startup analyst. Extract structured information from pitch decks.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.3,
    response_format: { type: 'json_object' },
  });

  const analysis = JSON.parse(
    response.choices[0].message.content || '{}'
  ) as PitchDeckAnalysis;

  return analysis;
}
```

---

## Phase 3: Secure Worker

### Step 3.1: Dockerfile

Create `docker/enclave.Dockerfile`:

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --production

# Copy worker code
COPY workers/ ./workers/
COPY lib/ ./lib/

# Security: Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Set environment
ENV NODE_ENV=production
ENV SECURE_MODE=true

# Run worker
CMD ["node", "workers/enclave-worker.js"]
```

### Step 3.2: Worker Entry Point

Create `workers/enclave-worker.ts`:

```typescript
import { decryptDocument, hashDocument } from '../lib/crypto/encryption';
import { extractTextFromPDF } from '../lib/parser/pdf';
import { analyzePitchDeck } from '../lib/ai/analyzer';
import { generateProof } from '../lib/proof/generator';

async function processJob(jobId: string) {
  // 1. Retrieve encrypted document (from API)
  const encryptedDoc = await retrieveEncryptedDocument(jobId);
  
  // 2. Reconstruct key (from ShadowVault)
  const key = await reconstructKey(jobId);
  
  // 3. Decrypt document
  const document = await decryptDocument(encryptedDoc, key);
  
  // 4. Verify document hash
  const documentHash = hashDocument(document);
  // Verify against stored hash...
  
  // 5. Extract text
  const text = await extractTextFromPDF(document);
  
  // 6. Run AI analysis
  const analysis = await analyzePitchDeck(text);
  
  // 7. Generate proof
  const proof = await generateProof(jobId, documentHash, analysis);
  
  // 8. Return result
  return { analysis, proof };
}

// Entry point
const jobId = process.argv[2];
if (!jobId) {
  console.error('Job ID required');
  process.exit(1);
}

processJob(jobId)
  .then((result) => {
    console.log(JSON.stringify(result));
    process.exit(0);
  })
  .catch((error) => {
    console.error('Error:', error);
    process.exit(1);
  });
```

---

## Phase 4: Frontend

### Step 4.1: Upload Page

Create `app/upload/page.tsx`:

```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export default function UploadPage() {
  const [file, setFile] = useState<File | null>(null);
  const [uploading, setUploading] = useState(false);
  const router = useRouter();

  const handleUpload = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!file) return;

    setUploading(true);
    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
      });

      const data = await response.json();
      router.push(`/dashboard/${data.jobId}`);
    } catch (error) {
      console.error('Upload failed:', error);
    } finally {
      setUploading(false);
    }
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">Upload Pitch Deck</h1>
      <form onSubmit={handleUpload} className="space-y-4">
        <input
          type="file"
          accept=".pdf,.docx,.pptx"
          onChange={(e) => setFile(e.target.files?.[0] || null)}
          className="block w-full"
        />
        <button
          type="submit"
          disabled={!file || uploading}
          className="px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50"
        >
          {uploading ? 'Uploading...' : 'Upload'}
        </button>
      </form>
    </div>
  );
}
```

### Step 4.2: Dashboard

Create `app/dashboard/page.tsx`:

```typescript
'use client';

import { useQuery } from '@tanstack/react-query';

export default function DashboardPage() {
  const { data: jobs } = useQuery({
    queryKey: ['jobs'],
    queryFn: async () => {
      const res = await fetch('/api/jobs');
      return res.json();
    },
  });

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">My Jobs</h1>
      <div className="space-y-4">
        {jobs?.map((job: any) => (
          <div key={job.id} className="border p-4 rounded">
            <h2 className="font-bold">{job.document.originalFilename}</h2>
            <p>Status: {job.status}</p>
            {job.result && (
              <a href={`/results/${job.id}`}>View Results</a>
            )}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Phase 5: Integration

### Step 5.1: Upload API

Create `app/api/upload/route.ts`:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import { writeFile } from 'fs/promises';
import { join } from 'path';
import { encryptDocument, generateKey, hashDocument } from '@/lib/crypto/encryption';
import { prisma } from '@/lib/db/prisma';
import { config } from '@/lib/config';

export async function POST(request: NextRequest) {
  try {
    const formData = await request.formData();
    const file = formData.get('file') as File;
    
    if (!file) {
      return NextResponse.json({ error: 'No file provided' }, { status: 400 });
    }

    const buffer = Buffer.from(await file.arrayBuffer());
    const documentHash = hashDocument(buffer);
    
    // Generate encryption key
    const key = generateKey();
    
    // Encrypt document
    const encrypted = await encryptDocument(buffer, key);
    
    // Store encrypted file
    const encryptedPath = join(config.storage.encryptedPath, `${documentHash}.enc`);
    await writeFile(encryptedPath, encrypted.encrypted);
    
    // Create document record
    const document = await prisma.document.create({
      data: {
        userId: 'user-id', // TODO: Get from auth
        originalFilename: file.name,
        mimeType: file.type,
        fileSize: file.size,
        encryptedFilePath: encryptedPath,
        documentHash,
      },
    });
    
    // Create job
    const job = await prisma.job.create({
      data: {
        documentId: document.id,
        userId: 'user-id', // TODO: Get from auth
        status: 'QUEUED',
      },
    });
    
    // TODO: Store key fragments in ShadowVault
    
    return NextResponse.json({
      jobId: job.id,
      documentHash,
      timestamp: new Date().toISOString(),
    });
  } catch (error) {
    console.error('Upload error:', error);
    return NextResponse.json(
      { error: 'Upload failed' },
      { status: 500 }
    );
  }
}
```

---

## Phase 6: Testing

### Step 6.1: Unit Tests

Create test files for each module:
- `lib/crypto/encryption.test.ts`
- `lib/parser/pdf.test.ts`
- `lib/ai/analyzer.test.ts`

### Step 6.2: Integration Tests

Test end-to-end flows:
- Upload → Process → Results
- Proof generation and verification

---

## Phase 7: Deployment

### Step 7.1: Prepare for Production

1. Set environment variables
2. Build Docker images
3. Set up database
4. Configure storage

### Step 7.2: Deploy

1. Deploy Next.js to Vercel
2. Deploy database (Supabase/Neon)
3. Set up Docker registry
4. Deploy workers

---

## Next Steps

1. **Read Research Papers** (see RESEARCH_PAPERS.md)
2. **Follow Implementation Guide**
3. **Test Incrementally**
4. **Iterate Based on Feedback**

Good luck building! 🚀

