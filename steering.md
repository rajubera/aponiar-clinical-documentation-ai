# Clinical Documentation AI - Steering Document
Strategic vision, product goals, and business requirements.

## 1. Product Overview
Build a Clinical Documentation AI web and mobile application that allows doctors to record or upload patient consultations, automatically generate structured clinical notes (SOAP format), allow manual entry of vitals and advice, edit notes, and export/share securely.

The system must be:
- Serverless-first (AWS Lambda preferred)
- Multi-tenant SaaS (multiple hospitals/doctors)
- Mobile-friendly
- Privacy-by-design
- Fully compliant with India DPDP Act 2023
- Architected with a separate compliance abstraction layer so it can later support HIPAA and GDPR

## 2. Target Users
**Primary users:**
- Doctor
- Clinic admin
- Hospital admin

**Secondary:**
- Patient (future phase)

## 3. Core Features

### 3.1 Authentication & Access Control
Support:
- Doctor account
- Organization account

Role-based access control:
- Doctor
- Admin
- Super admin

Security requirements:
- JWT authentication
- Session expiration
- Device tracking (optional)

### 3.2 Dashboard
Doctor dashboard showing:
- Recent consultations
- Search patient
- Create new consultation
- Draft vs completed notes

### 3.3 Create Consultation Workflow
**Step 1: Enter/select patient**
Fields:
- Patient name
- Age
- Gender
- Phone (optional)
- Patient ID (optional)

**Step 2: Record or upload audio**
Support:
- Browser recording
- Mobile recording
- Audio file upload

### 3.4 Speech-to-Text Layer
System sends audio to transcription service with abstract provider interface for flexibility.

### 3.5 AI Clinical Note Generation
System generates structured note using LLM.

Required formats:
- SOAP note (Subject, Objective, Assessment, Plan)
- Discharge summary (optional)
- Prescription draft (optional)
- Must allow regeneration

### 3.6 Structured Vitals Entry
Doctor can manually enter:
- Heart rate
- Blood pressure
- Temperature
- SpO2
- Respiratory rate
- Weight
- Height

### 3.7 Advice and Plan Entry
Doctor can add:
- Free text advice
- Medicine list

### 3.8 Clinical Note Editor
Editable rich text editor with:
- Auto save
- Version history
- Audit log

### 3.9 Export and Sharing
Export options:
- PDF
- Print
- Copy text

Future:
- Send to EMR via API

### 3.10 Consultation History
Doctor can view:
- Previous visits
- Vitals history
- Previous notes

## 4. Compliance Requirements - DPDP Act 2023

System must implement:
- Consent management
- Purpose limitation
- Data minimization
- Right to access
- Right to correction
- Right to erasure
- Data retention policy
- Breach logging

### 4.1 Consent Management
Before recording consultation:
- Doctor must confirm consent obtained
- Store consent status, timestamp, and method

### 4.2 Data Encryption
**Encryption in transit:** HTTPS TLS 1.2+

**Encryption at rest:** AES-256 for:
- Audio
- Transcript
- Notes
- Patient data

### 4.3 Data Access Control
- Role-based access control
- Organization-level isolation
- Doctors can access only their organization data

### 4.4 Audit Logging
Log every access event:
- User ID
- Resource accessed
- Timestamp
- Action

### 4.5 Data Retention System
Configurable retention policy example:
- Delete audio after 30 days
- Keep notes indefinitely

### 4.6 Data Deletion System
Allow deletion request:
- Delete audio
- Delete transcripts
- Delete notes
- Delete patient data
- Track deletion log

### 4.7 Data Localization
Store Indian user data in India region AWS.

## 5. HIPAA / GDPR Future Readiness

Design system to support:
- BAA providers
- PHI tagging
- Data anonymization
- Right to portability
- Right to be forgotten

Data classification tags:
- PHI
- PII
- NonSensitive

## 6. Multi-Tenant Architecture

Each organization must have isolated data.

Entities:
- Organization
- User
- Patient
- Consultation

All records linked to organizationId.
Strict isolation enforced at API layer.

## 7. Privacy-by-Design Principles
- Collect minimum data required
- No unnecessary storage
- Allow anonymization

## 8. MVP Scope

Must include:
- Audio recording
- AI note generation
- Vitals entry
- Editor
- PDF export
- Consent capture
- Audit logging

## 9. Future Features
- EMR integration
- ICD coding generation
- Prescription generation
- Patient mobile app

## 10. Business-Critical Requirements

✓ Multi-tenant from day one
✓ Compliance abstraction layer mandatory
✓ Encryption mandatory
✓ Audit logging mandatory

## 11. Performance Requirements
- Generate note in under 10 seconds
- Support 10,000 concurrent users scalable

## 12. Recommended Tech Stack

**Frontend:**
- React
- React Native
- Tailwind

**Backend:**
- AWS Lambda
- Node.js
- Serverless framework

**Database:**
- MongoDB Atlas

**Storage:**
- AWS S3

**AI:**
- OpenAI or Claude

**Auth:**
- AWS Cognito
