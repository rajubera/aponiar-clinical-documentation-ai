# Clinical Documentation AI - Technical Design Document
Architecture, implementation patterns, and technical specifications.

## 1. Compliance-First Architecture (CRITICAL)

Create a dedicated layer called:
**Compliance Layer**

This must sit between:
- Application Layer
- Storage / AI / Logs

**Purpose:**
- Centralize privacy and compliance logic
- Enable multi-region compliance support
- Simplify compliance rule updates

## 2. Compliance Abstraction Layer Design

Create modular compliance interfaces:

### 2.1 ConsentService
```
recordConsent()
verifyConsent()
revokeConsent()
getConsentHistory()
```

### 2.2 EncryptionService
```
encrypt()
decrypt()
```

### 2.3 AuditService
```
logAccess()
logAction()
queryAuditLog()
```

### 2.4 RetentionService
```
scheduleDeletion()
executeDeletion()
getRetentionPolicy()
```

### 2.5 AccessControlService
```
checkAccess()
enforceOrgIsolation()
validateRole()
```

### 2.6 DataDeletionService
```
deleteUserData()
deleteConsultationData()
trackDeletionLog()
```

### 2.7 ComplianceConfig
Centralized configuration for compliance provider implementations.

## 3. Compliance Layer Folder Structure

```
backend/
├── compliance/
│   ├── services/
│   │   ├── consent.service.ts
│   │   ├── encryption.service.ts
│   │   ├── audit.service.ts
│   │   ├── retention.service.ts
│   │   ├── access-control.service.ts
│   │   └── data-deletion.service.ts
│   ├── interfaces/
│   │   ├── consent.interface.ts
│   │   ├── encryption.interface.ts
│   │   ├── audit.interface.ts
│   │   ├── retention.interface.ts
│   │   ├── access-control.interface.ts
│   │   └── data-deletion.interface.ts
│   ├── providers/
│   │   └── dpdp.provider.ts
│   ├── future/
│   │   ├── hipaa.provider.ts
│   │   └── gdpr.provider.ts
│   └── compliance.config.ts
├── ai/
├── storage/
└── logs/
```

## 4. Key Compliance Principle

**Compliance logic must NEVER be mixed directly into business logic.**

Always use compliance layer for:
- Data access decisions
- Encryption/decryption
- Audit logging
- Data retention and deletion
- Consent verification

## 5. Backend Architecture

### 5.1 API Layer
- AWS API Gateway with rate limiting and request validation

### 5.2 Compute
- AWS Lambda for all business logic
- Serverless framework for deployment

### 5.3 Storage Layer
- S3 for encrypted audio files
- MongoDB Atlas for structured data with encryption at rest

### 5.4 Queue Processing
- SQS for asynchronous tasks (transcription, note generation, deletion jobs)

### 5.5 Secrets Management
- AWS Secrets Manager for API keys, database credentials

### 5.6 Monitoring & Logging
- CloudWatch for logs and metrics
- CloudWatch Alarms for critical events

## 6. Database Schema (MongoDB)

### Organization
```
{
  _id: ObjectId
  name: String
  organizationId: String (unique)
  createdAt: Date
  updatedAt: Date
  region: String (for data localization)
  complianceProvider: String (DPDP, HIPAA, GDPR)
}
```

### User
```
{
  _id: ObjectId
  organizationId: String
  role: Enum (Doctor, Admin, SuperAdmin)
  email: String (encrypted)
  name: String (encrypted)
  createdAt: Date
  lastLogin: Date
}
```

### Patient
```
{
  _id: ObjectId
  organizationId: String
  name: String (encrypted)
  age: Number
  gender: String
  phone: String (encrypted, optional)
  patientId: String (optional, encrypted)
  createdAt: Date
  dataClassification: Enum (PHI, PII, NonSensitive)
}
```

### Consultation
```
{
  _id: ObjectId
  organizationId: String
  doctorId: String
  patientId: String
  audioUrl: String (encrypted S3 URL)
  transcript: String (encrypted)
  vitals: {
    heartRate: Number
    bloodPressure: String
    temperature: Number
    spO2: Number
    respiratoryRate: Number
    weight: Number
    height: Number
  }
  note: {
    subject: String
    objective: String
    assessment: String
    plan: String
  }
  advice: {
    freeText: String
    medicines: [
      {
        name: String
        dose: String
        frequency: String
        duration: String
      }
    ]
  }
  consent: {
    status: Enum (obtained, pending, revoked)
    timestamp: Date
    method: String
  }
  status: Enum (draft, completed)
  createdAt: Date
  updatedAt: Date
  deletionScheduledAt: Date (null unless scheduled)
}
```

### AuditLog
```
{
  _id: ObjectId
  organizationId: String
  userId: String
  action: String
  resource: String
  resourceId: String
  resourceType: String
  timestamp: Date
  ipAddress: String
  result: Enum (success, failure)
  details: Object
}
```

### ConsentLog
```
{
  _id: ObjectId
  organizationId: String
  patientId: String
  doctorId: String
  consultationId: String
  consentStatus: Enum (obtained, revoked)
  timestamp: Date
  method: String
  expiresAt: Date (optional)
}
```

### DataDeletionLog
```
{
  _id: ObjectId
  organizationId: String
  userId: String (who requested deletion)
  targetType: String (patient, consultation, user)
  targetId: String
  status: Enum (scheduled, in_progress, completed, failed)
  requestedAt: Date
  completedAt: Date (null until complete)
  reason: String
}
```

## 7. AI Integration Layer

Create abstraction:
```typescript
interface AIProvider {
  transcribeAudio(audioBuffer: Buffer): Promise<string>
  generateClinicalNote(transcript: string): Promise<ClinicalNote>
}
```

Implementations:
- OpenAI Whisper (transcription)
- Claude / GPT-4 (clinical note generation)
- Self-hosted option (future)

Providers must be switched via ComplianceConfig.

## 8. Transcription Provider Abstraction

```typescript
interface TranscriptionProvider {
  transcribe(audioUrl: string): Promise<string>
  getTranscriptStatus(jobId: string): Promise<TranscriptionStatus>
}
```

Implementation examples:
- OpenAI Whisper
- AWS Transcribe Medical
- Self-hosted (future)

## 9. Security Guidelines

✓ Never expose PHI in logs
✓ Mask sensitive data in logs (emails indexed but values masked)
✓ Use signed URLs for audio access with 15-minute expiration
✓ Use short-lived JWT tokens (15 minutes)
✓ Refresh tokens stored in secure cookies (httpOnly, Secure flags)
✓ Implement rate limiting on all API endpoints
✓ Validate all inputs at API boundary
✓ Use CORS whitelist for frontend origins
✓ TLS 1.2+ for all data in transit
✓ IP whitelisting optional for high-security deployments

## 10. Frontend Requirements

Technology:
- React with TypeScript
- React Native for mobile
- Tailwind CSS for styling

Screens:
- Login/Authentication
- Dashboard with consultation list
- New consultation creation
- Audio recording interface
- Vitals entry form
- AI-generated note display
- Clinical note editor
- Consultation history view
- Patient search/lookup
- Settings and preferences

## 11. API Endpoints (Example)

```
POST   /api/v1/consultations              - Create consultation
GET    /api/v1/consultations/:id          - Get consultation
PUT    /api/v1/consultations/:id          - Update consultation
DELETE /api/v1/consultations/:id          - Delete (soft) consultation

POST   /api/v1/audio/presigned-url        - Get S3 upload URL
POST   /api/v1/transcription              - Trigger transcription
GET    /api/v1/transcription/:jobId       - Get transcription status

POST   /api/v1/notes/generate             - Generate clinical note
PUT    /api/v1/notes/:consultationId      - Update note

POST   /api/v1/consent/record             - Record consent
GET    /api/v1/consent/:consultationId    - Verify consent
DELETE /api/v1/consent/:consultationId    - Revoke consent

GET    /api/v1/audit-logs                 - Query audit logs (Admin)
POST   /api/v1/deletion-requests          - Request data deletion
GET    /api/v1/deletion-status/:requestId - Check deletion status

GET    /api/v1/patients                   - Search patients
POST   /api/v1/patients                   - Create patient
```

## 12. Deployment Architecture

### Infrastructure as Code
- Terraform for AWS resource provisioning
- CloudFormation templates for Lambda deployments

### Environment Isolation
```
dev/     - Development environment, auto-deploy on commits
staging/ - Pre-production, manual deployment
prod/    - Production, change control required
```

Each environment has:
- Separate AWS account or VPC
- Isolated MongoDB database
- Separate S3 buckets
- Dedicated IAM roles

## 13. Lambda Function Architecture

Functions organized by domain:
```
functions/
├── auth/
│   ├── login.ts
│   ├── logout.ts
│   └── refresh-token.ts
├── consultations/
│   ├── create.ts
│   ├── read.ts
│   ├── update.ts
│   └── delete.ts
├── notes/
│   ├── generate.ts
│   ├── edit.ts
│   └── export.ts
├── transcription/
│   ├── submit.ts
│   └── webhook.ts
├── audit/
│   └── query.ts
└── compliance/
    ├── consent-handler.ts
    ├── deletion-handler.ts
    └── retention-handler.ts
```

Each Lambda includes:
- Input validation
- Compliance layer invocation
- Error handling with proper HTTP codes
- Structured logging
- Metrics emission

## 14. Logging Strategy

All logs structured as JSON:
```json
{
  "timestamp": "2026-03-02T10:30:00Z",
  "level": "info",
  "service": "consultations",
  "function": "generateNote",
  "organizationId": "org_xxx",
  "userId": "[MASKED]",
  "action": "note_generation",
  "resourceType": "consultation",
  "resourceId": "cons_xxx",
  "duration": 8500,
  "result": "success"
}
```

Sensitive fields (emails, names, phone) masked in all logs.
PHI never logged directly.

## 15. Monitoring & Alerting

**CloudWatch Metrics:**
- Lambda execution duration
- Error rates by function
- Transcription job completion time
- Note generation latency
- API gateway request count & errors
- Compliance violation attempts
- Unencrypted data access attempts

**Critical Alerts:**
- Lambda error rate > 1%
- Note generation > 15 seconds
- Deletion job failures
- Unauthorized API access attempts
- Encryption service failures

## 16. Data Flow Architecture

1. **Consultation Recording**
   - User records audio via frontend
   - Audio uploaded to S3 via presigned URL
   - Notification sent to transcription queue (SQS)

2. **Transcription Processing**
   - Lambda picks up from SQS
   - Calls transcription provider
   - Stores encrypted transcript in MongoDB
   - Queues note generation task

3. **Note Generation**
   - Lambda receives transcript
   - Calls AI provider (Claude/GPT-4)
   - Stores encrypted note in MongoDB
   - Updates consultation status

4. **Data Access**
   - All queries go through DecryptionService
   - Audit log recorded via AuditService
   - Organization isolation enforced by AccessControlService

5. **Data Deletion**
   - Deletion request submitted
   - Queued for async processing
   - Compliance layer validates rights
   - Sequential deletion: audio → transcript → notes → metadata
   - Deletion log immutably stored

## 17. Performance Optimization

**Caching:**
- API Gateway caching for patient list (5 min TTL)
- Redis for session data (future, if needed)

**Database Indexing:**
- organizationId + createdAt (consultation queries)
- userId + timestamp (audit logs)
- organizationId + patientId (patient consultations)

**Async Processing:**
- Long-running tasks (transcription, note generation) processed via SQS
- Frontend polls for completion via status endpoint
- WebSocket future enhancement for real-time updates

## 18. Version Control & API Versioning

- API versions through URL path: `/api/v1/`, `/api/v2/`
- Backward compatibility maintained for 2 major versions
- Deprecation warnings in response headers

## 19. Error Handling Standards

All API errors return structured response:
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Patient age is required",
    "details": {
      "field": "patient.age",
      "reason": "required"
    }
  }
}
```

HTTP Status Codes:
- 400: Bad Request (validation errors)
- 401: Unauthorized (auth required)
- 403: Forbidden (insufficient permissions)
- 404: Not Found
- 429: Too Many Requests (rate limited)
- 500: Internal Server Error
- 503: Service Unavailable

## 20. Testing Strategy

**Unit Tests:**
- Compliance layer services (100% coverage targeted)
- Data validation logic
- Business logic calculations

**Integration Tests:**
- API endpoint tests with real database
- Compliance layer integration
- Encryption/decryption roundtrips

**E2E Tests:**
- Full consultation workflow
- Multi-tenant isolation
- Compliance audit trail

**Load Tests:**
- 10,000 concurrent users simulation
- Note generation under load
- Database performance under peak load
