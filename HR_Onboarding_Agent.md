# HR Onboarding Agent - System Design Document

**Document Version Control**

| Version | Date       | Author   | Change Description                                      |
|---------|------------|----------|--------------------------------------------------------|
| 1.0     | 2025-11-25 | Md Sahil | Initial draft, including introduction and system overview sections. |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Recruitment Process Flow](#2-recruitment-process-flow)
3. [System Overview](#3-system-overview)
4. [Infrastructure Architecture](#4-infrastructure-architecture)
5. [Technology Stack](#5-technology-stack)
6. [User Roles & Access Hierarchy (RBAC)](#6-user-roles--access-hierarchy-rbac)
7. [Architecture Design](#7-architecture-design)
8. [Module Descriptions](#8-module-descriptions)
9. [Data Design](#9-data-design)
10. [Deployment Architecture](#10-deployment-architecture)
11. [Compliance & Data Retention](#11-compliance--data-retention)
12. [Risks & Mitigations](#12-risks--mitigations)
13. [Success Metrics & KPIs](#13-success-metrics--kpis)
14. [HR Recruitment Process Flow](#14-hr-recruitment-process-flow)

---

## 1. Introduction

### 1.1 Purpose

The **HR Onboarding Agent Project** aims to digitize and automate the end-to-end **recruitment and onboarding process** for Sampurna Financial Services Pvt. Ltd. (SFSPL).

#### ❌ Previous Challenges

**Manual Process Issues:**
- ❌ Data loss and duplication
- ❌ Delayed and unstructured communication
- ❌ Limited visibility on candidate progress
- ❌ No track of interview lifecycle
- ❌ No scope for analysis
- ❌ Manual handling via WhatsApp, Excel sheets, and emails

#### ✅ Solution Objectives

**Centralized Digital System Benefits:**
- ✅ Secure storage of all candidate and interview data
- ✅ Automated screening, evaluation, scheduling, and tracking
- ✅ Real-time visibility of onboarding progress
- ✅ Reduced manual dependency and enhanced HR efficiency
- ✅ Interview process and quality analysis

#### 📑 Problem Statement

The current employee onboarding process is highly manual and lacks digital structure. Candidates send CVs via WhatsApp, HR contacts them for preliminary questions, and interviews are conducted offline. Information is not systematically recorded, resulting in:

- No centralized or organized database
- Only mandatory details entered into HRMS
- Significant data loss and inefficiency
- Difficulty tracking and analyzing recruitment trends

### 1.2 🎯 Scope

#### ✅ In Scope
- Candidate registration & digital forms (Form-1 to Form-4)
- Interview scheduling & panel allocation
- Profile, document & KYC management
- Onboarding checklist & task tracking
- Email & WhatsApp notifications (via n8n)
- PostgreSQL centralized database
- Multilingual interface (English, Hindi, Bengali)
- Analysis & Reporting

#### ❌ Out of Scope
- Full automation for background verification
- Direct integration with Octane
- Auto mail from HR mailbox
- Auto selection of candidates

---

## 2. Recruitment Process Flow

### Recruitment Process Flow Diagram

```mermaid
graph TD
    A["🎯 Vacancy<br/>Identification"] --> B["📱 WhatsApp<br/>Campaign"]
    B --> C["📝 CV<br/>Submission"]
    C --> D["📋 Form-1<br/>Basic Details"]
    D --> E{"✅ Eligibility<br/>Check"}
    E -->|Fail| F["❌ Rejected"]
    E -->|Pass| G["📅 Interview<br/>Scheduled"]
    G --> H["🏢 Interview<br/>Day"]
    H --> I["📋 Form-2<br/>Details"]
    I --> J["🎤 Interview<br/>Evaluation"]
    J --> K{"Selection<br/>Decision"}
    K -->|Selected| L["📄 Offer<br/>Letter"]
    K -->|Rejected| F
    L --> M["✅ Form-4<br/>Onboarding"]
    M --> N["🎉 Onboarding<br/>Complete"]
    
    style A fill:#e3f2fd
    style B fill:#e3f2fd
    style C fill:#e3f2fd
    style D fill:#e3f2fd
    style E fill:#fff3e0
    style F fill:#ffcdd2
    style G fill:#f3e5f5
    style H fill:#f3e5f5
    style I fill:#f3e5f5
    style J fill:#f3e5f5
    style K fill:#fff3e0
    style L fill:#c8e6c9
    style M fill:#c8e6c9
    style N fill:#a5d6a7
```

**[Diagram 1: Complete Recruitment Flow]**

---

### 2.1 Data Flow Diagram

#### Level-0: System Context

```mermaid
graph LR
    EXT["📱 External Systems<br/>WhatsApp, Email<br/>Octane HRMS"]
    
    HR_USR["👥 HR Users<br/>Recruiters<br/>Managers"]
    
    SYSTEM["🎯 HR Onboarding<br/>Agent System"]
    
    CAND_USR["👤 Candidates"]
    
    EXT <-->|Data Exchange| SYSTEM
    HR_USR <-->|Manage| SYSTEM
    SYSTEM <-->|Engage| CAND_USR
    
    style SYSTEM fill:#fff9c4
    style HR_USR fill:#e8f5e9
    style CAND_USR fill:#e3f2fd
    style EXT fill:#fce4ec
```

**[Diagram 2: Data Flow Level-0]**

#### Level-1: Detailed Processes

- **Process 1:** Candidate Registration (Forms 1-4)
- **Process 2:** Interview Scheduling & Management
- **Process 3:** Decision & Notification
- **Process 4:** Onboarding Task Tracking
- **Process 5:** Analytics & Reporting

---

### 2.2 Technology Stack and Architecture

**Core Technology Stack:**

```mermaid
graph TB
    subgraph "Frontend Layer"
        CP["📱 Candidate Portal<br/>Streamlit 8501"]
        HP["📊 HR Dashboard<br/>Streamlit 8502"]
    end
    
    subgraph "Backend Layer"
        FA["⚡ FastAPI<br/>Port 8000"]
        N8N["🔄 n8n Automation"]
    end
    
    subgraph "Data Layer"
        DB["🗄️ PostgreSQL<br/>Supabase"]
        BLOB["📦 Supabase Storage<br/>Resumes, KYC"]
    end
    
    subgraph "Infrastructure"
        DOCKER["🐳 Docker Compose<br/>Hostinger VPS"]
        GIT["📘 GitHub<br/>CI/CD"]
    end
    
    CP -->|HTTPS| FA
    HP -->|HTTPS| FA
    FA -->|Query| DB
    FA -->|Upload| BLOB
    FA -->|Webhooks| N8N
    DOCKER -->|Manages| CP
    DOCKER -->|Manages| HP
    DOCKER -->|Manages| FA
    GIT -->|Triggers| DOCKER
    
    style CP fill:#e3f2fd
    style HP fill:#f3e5f5
    style FA fill:#fff3e0
    style DB fill:#c8e6c9
    style BLOB fill:#fce4ec
    style N8N fill:#f1f8e9
```

**[Diagram 3: Technology Stack Architecture]**

---

### 2.3 👥 Target Audience

| Role | Responsibilities |
|------|------------------|
| HR & Business Users | Process owners, approvers, end users |
| System Architects | Review system components and overall architecture |
| Developers | Implement modules based on design |
| Database Administrators | Maintain PostgreSQL/Supabase schema and data integrity |
| Security & Compliance Team | Verify encryption, access control, and data retention |
| IT & DevOps Team | Manage deployment, monitoring, and version control |
| Management / Stakeholders | Approve scope, KPIs, and release milestones |

---

## 3. 🌐 System Overview

### 3.1 System Overview

The **HR Onboarding Agent** serves as a **web-based digital platform** that connects candidates, recruiters, and HR teams in one centralized ecosystem.

**Key Platform Features:**
- Streamlit-based user interfaces for candidates and HR
- FastAPI backend for business logic and API services
- PostgreSQL database for centralized data storage
- n8n automation for notifications and workflow triggers
- Supabase storage for documents and file management

**[Diagram 4: System Architecture Overview]**

```mermaid
graph TB
    CP["👥 Candidate Portal<br/>Streamlit 8501"]
    HP["📊 HR Dashboard<br/>Streamlit 8502"]
    API["⚡ FastAPI<br/>Backend<br/>Port 8000"]
    DB["🗄️ PostgreSQL<br/>Supabase"]
    STORAGE["📦 Supabase<br/>Storage"]
    N8N["🔄 n8n<br/>Automation"]
    
    CP -->|HTTPS| API
    HP -->|HTTPS| API
    API -->|CRUD| DB
    API -->|Files| STORAGE
    API -->|Webhooks| N8N
    
    style CP fill:#e3f2fd
    style HP fill:#f3e5f5
    style API fill:#fff3e0
    style DB fill:#c8e6c9
    style STORAGE fill:#fce4ec
    style N8N fill:#f1f8e9
```

### 3.2 🎯 Key Objectives

| Category | Objective | Outcome |
|----------|-----------|---------|
| Business | Eliminate manual WhatsApp/Excel-based tracking | Improved process reliability |
| Business | Reduce HR workload through automation | 60% less manual data handling |
| Business | Interview efficiency analysis | Helps understand interview quality |
| Technical | Store all data digitally | PostgreSQL-based centralized DB |
| Technical | Integrate automated notifications | Improved candidate communication |
| Operational | Track onboarding progress | Transparent HR reporting dashboard |

---

## 4. Infrastructure Architecture

### 4.1 Hosting Environment

The HR Onboarding Agent is hosted on a **Hostinger VPS** and follows a **Docker-based deployment model** for isolation, scalability, and simplified maintenance.

| Component | Environment | Purpose |
|-----------|-------------|---------|
| FastAPI Backend | VPS (Ubuntu 22.04) | Core business logic and API services |
| Streamlit Frontend | VPS | HR and Candidate user interfaces |
| PostgreSQL (Supabase) | VPS | Centralized database for recruitment and onboarding |
| n8n Automation | Internal Docker Service | Notification handling (Email, SMS, WhatsApp) |
| Blob Storage (Supabase) | VPS | Secure file storage for resumes |

### 4.2 🔒 Security & Networking

#### Security Measures
- ✅ HTTPS enforced (TLS 1.2+) with Let's Encrypt SSL
- ✅ Firewall rules restrict access to ports 80, 443, 22
- ✅ Role-based access control (RBAC)
- ✅ Daily backups with 30-day retention

#### Network Architecture
- API and database communicate via private network
- TLS at each application layer
- JWT authentication
- Optional DB Row-Level Security (RLS)

### 4.3 📐 Logical Topology

```mermaid
graph TB
    subgraph "Client Layer"
        CP["👥 Candidate Portal<br/>Streamlit 8501<br/>HTTPS"]
        HP["📊 HR Dashboard<br/>Streamlit 8502<br/>HTTPS"]
    end
    
    subgraph "API Layer"
        API["⚡ FastAPI<br/>Port 8000 HTTPS<br/>RBAC, Validation"]
    end
    
    subgraph "Data Layer"
        DB["🗄️ PostgreSQL<br/>Supabase<br/>Private Network"]
        STORAGE["📦 Supabase Storage<br/>Signed URLs"]
    end
    
    subgraph "Automation"
        N8N["🔄 n8n<br/>Private<br/>Webhooks"]
    end
    
    subgraph "Security"
        TLS["🔐 TLS 1.3<br/>JWT Auth<br/>Firewall"]
    end
    
    CP -->|HTTPS| API
    HP -->|HTTPS| API
    API -->|Private| DB
    API -->|Private| STORAGE
    API -->|Private| N8N
    TLS -->|Protects| API
    
    style CP fill:#e3f2fd
    style HP fill:#f3e5f5
    style API fill:#fff3e0
    style DB fill:#c8e6c9
    style STORAGE fill:#fce4ec
    style N8N fill:#f1f8e9
    style TLS fill:#ffcdd2
```

**[Diagram 5: Logical Topology]**

---

## 5. Technology Stack

### 5.1 Stack Components

| Layer | Primary Technology | Why Chosen | Notes |
|-------|-------------------|-----------|-------|
| Frontend & UI | Streamlit (Python) | Fast internal apps, form-heavy UI, minimal JS | Candidate Portal, HR Dashboard |
| Backend | FastAPI (Python) & n8n | High-performance async APIs, Pydantic validation, OpenAPI | Service layer + auth + business rules |
| Database | PostgreSQL (Supabase) | ACID, JSONB, solid indexing, easy backups | RLS-ready; UUID keys; audit tables |
| Storage | Supabase Storage | Signed URLs & lifecycle policies | Resumes, KYC, offer PDFs |
| Containerization | Docker Compose | Environment parity, reproducible builds | DEV/UAT/PROD |
| BI & Dashboards | Python (pandas + Plotly) | Quick analytics & exports | Runs inside HR Dashboard |
| Version Control | Git / GitHub | PR reviews, issues, Actions CI | Branch policy: main protected |

### 5.2 Component-to-Tool Mapping

```mermaid
graph LR
    CP["Candidate Portal"] --> ST["Streamlit"]
    HP["HR Dashboard"] --> ST
    API["Backend API"] --> FA["FastAPI"]
    AUTH["Authentication"] --> JWT["JWT Tokens"]
    AUTO["Automation"] --> N8N["n8n"]
    DB["Database"] --> PG["PostgreSQL"]
    STORAGE["File Storage"] --> SB["Supabase"]
    
    style ST fill:#e3f2fd
    style FA fill:#fff3e0
    style JWT fill:#ffe0b2
    style N8N fill:#f1f8e9
    style PG fill:#c8e6c9
    style SB fill:#fce4ec
```

### 5.3 🔒 Security Baselines

#### Authentication
- JWT (access/refresh tokens)
- Bcrypt passwords for HR users
- OTP for candidates

#### Data Protection
- TLS on all endpoints
- Signed URLs for files
- Least-privilege DB roles

### 5.4 CI/CD Overview (GitHub)

```
Lint → Tests → Build Docker Images
         ↓
Push to Registry → Remote Docker Compose Up
         ↓
Alembic Upgrade → Smoke Tests → Production
```

**Branches:** `feature` → `PR` → `main` (protected)

### 5.5 🔮 Future Enhancements

#### ✨ Planned Features
- Streamlit component polish
- WhatsApp chatbot with map facilities
- Maximize automation

#### 🎯 Strategic Goals
- Ensure Security & Compliance
- Make it sellable as product

---

## 6. User Roles & Access Hierarchy (RBAC)

### 6.1 Role Catalog (Least-Privilege)

| Role | Responsibilities |
|------|------------------|
| Super Admin | User provisioning, environment/config toggles |
| HR Manager | Approvals, final decisions, reports |
| Recruiter | Intake, screening, schedule interviews, draft updates |
| Interviewer | View assigned candidates, submit Form-3 only |
| Compliance/Audit | Read-only to audit logs/reports |
| Candidate | Own profile/forms/documents; track status, ask questions through chatbot |

### 6.2 🔐 Permission Snapshot (CRUD + Approvals)

```mermaid
graph TD
    SA["👑 Super Admin<br/>User Provisioning<br/>Config Management"]
    HRM["👔 HR Manager<br/>Approvals<br/>Final Decisions<br/>Reports"]
    REC["🔍 Recruiter<br/>Screening<br/>Scheduling"]
    INT["👨‍💼 Interviewer<br/>Form-3 Only<br/>Submit Evaluations"]
    AUDIT["📋 Compliance/Audit<br/>Read-Only Access"]
    CAND["👤 Candidate<br/>Self-Profile<br/>Status Tracking"]
    
    SA -->|Manages| HRM
    SA -->|Manages| REC
    SA -->|Manages| INT
    HRM -->|Reviews| REC
    HRM -->|Supervises| INT
    HRM -->|Communicates| CAND
    REC -->|Engages| CAND
    INT -->|Evaluates| CAND
    AUDIT -->|Monitors All|SA
    
    style SA fill:#ffebee
    style HRM fill:#fff3e0
    style REC fill:#e3f2fd
    style INT fill:#f3e5f5
    style AUDIT fill:#fce4ec
    style CAND fill:#e8f5e9
```

### 6.3 📋 Data Scope & Session Policy

| Aspect | Policy |
|--------|--------|
| Scope | Candidate=self; Interviewer=assigned_only; Recruiter=branch/assigned; HR Manager=department/global |
| HR Sessions | 8h session (2h idle timeout) + optional MFA for managers |
| Candidate Sessions | 24h session with OTP authentication |
| RLS (Optional) | Policies by candidate_id, branch, or assignment in Supabase |

---

## 7. Architecture Design

### 7.1 🧩 Components

```mermaid
graph TB
    subgraph "Core Modules"
        CM["📝 Candidate Module<br/>Registration, Forms 1-4"]
        HM["👔 HR Module<br/>Screening, Scheduling"]
        IM["🎤 Interview Module<br/>QR Check-in, Form-3"]
        OM["✅ Onboarding Module<br/>Checklist, Tasks"]
    end
    
    subgraph "Support Modules"
        AUTH["🔐 Auth Module<br/>JWT, OTP, RBAC"]
        NOTIF["📧 Notification Module<br/>Email, SMS, WhatsApp"]
        ANALYTICS["📊 Analytics Module<br/>Dashboards, Reports"]
        AUDIT["📋 Audit Module<br/>Logging, Compliance"]
    end
    
    CM -->|Uses| AUTH
    HM -->|Uses| AUTH
    IM -->|Uses| AUTH
    OM -->|Uses| AUTH
    
    CM -->|Triggers| NOTIF
    HM -->|Triggers| NOTIF
    IM -->|Triggers| NOTIF
    
    CM -->|Logs| AUDIT
    HM -->|Logs| AUDIT
    IM -->|Logs| AUDIT
    OM -->|Logs| AUDIT
    
    style CM fill:#e3f2fd
    style HM fill:#fff3e0
    style IM fill:#f3e5f5
    style OM fill:#c8e6c9
    style AUTH fill:#ffe0b2
    style NOTIF fill:#f1f8e9
    style ANALYTICS fill:#eceff1
    style AUDIT fill:#fce4ec
```

### 7.2 🔄 Component Interactions

| Step | Interaction |
|------|-------------|
| Step 1 | Candidate opens Streamlit Form → Provides mobile number → FastAPI verifies conditions → Redirects to form |
| Step 2 | Forms post JSON → FastAPI validates → Writes to PostgreSQL |
| Step 3 | File uploads → FastAPI requests signed URL → Streamlit uploads to Supabase Storage |
| Step 4 | HR actions (screening, scheduling, decisions) → FastAPI/n8n updates DB → Triggers automation webhooks |
| Step 5 | Dashboards/analytics → Streamlit queries DB views for read-only data |

### 7.3 📊 Sequence Diagram (Happy Path)

```mermaid
sequenceDiagram
    actor C as Candidate
    participant WA as WhatsApp Bot
    participant API as FastAPI Backend
    participant DB as PostgreSQL
    participant N8N as n8n Automation
    participant HR as HR Dashboard
    
    C->>WA: "Hi" Message
    WA->>API: Log initiation
    API->>DB: Create candidate session
    WA->>C: Send Form-1 link
    C->>API: Submit Form-1
    API->>DB: Upsert personal_details
    API->>N8N: Webhook: Acknowledgment
    N8N->>C: WhatsApp Confirmation
    
    HR->>API: Review & Schedule Interview
    API->>DB: Update application_status
    API->>N8N: Webhook: Interview Invite
    N8N->>C: WhatsApp Invite
    
    C->>API: Submit Form-2
    API->>DB: Insert addresses, qualifications
    
    C->>API: Scan QR at Venue
    API->>DB: Log check-in
    
    HR->>API: Submit Form-3 Evaluation
    API->>DB: Insert interview_evaluations
    
    HR->>API: Mark Selected
    API->>DB: Update status = 'selected'
    API->>N8N: Webhook: Selection Notification
    N8N->>C: Offer Letter Sent
```

---

## 8. Module Descriptions

### 📝 8.1 Candidate Management

**What it does:**
Mobile number based login, Form-1 (basic), Form-2 (details), document uploads, status tracking.

**Key Flows:**
1. WhatsApp "Hi" → Form-1 link → Eligibility check (number + 30-day rule)
2. Form-1 upsert personal_details + insert candidate_applications
3. Form-2 writes addresses, qualifications, skills, documents

**UI:** Streamlit forms + status timeline

---

### 👔 8.2 HR Management

**What it does:**
Screening, shortlist/reject, interview scheduling (date/time/location), offer decision gate.

**Notes:**
Scheduling updates are the trigger-point for automation to send invites.

---

### 🎤 8.3 Interview Management

**What it does:**
QR check-in at venue, Form-3 evaluation (ratings + remarks).

**Rules:**
Interviewer loads Form-3 by candidate mobile; writes interview_evaluations, updates application status.

---

### 📄 8.4 Offer & Final Decision

**What it does:**
Post-selection but before joining: Creates checklist, captures joining/KYC documents, verifies completion.

---

### ✅ 8.5 Onboarding Tasks

**What it does:**
Post-selection but before joining: Creates checklist, captures joining/KYC documents, verifies completion.

**Triggers:**
- Candidate selection completion
- Form-4 submission initiation

**Mechanism:**
FastAPI creates onboarding checklist → n8n sends reminders → Streamlit tracks completion

---

### 📧 8.6 Notifications (via Automation)

**Triggers:**
- Form-1 acknowledgment
- Interview invite/reminders
- Decision notice
- Onboarding welcome

**Mechanism:**
FastAPI emits webhooks → n8n automation sends WhatsApp/Email/SMS; delivery logged.

---

### 📊 8.7 Analytics & Reporting

**Dashboards:**
- Pipeline funnel (candidates at each stage)
- Time-to-hire metrics
- Source effectiveness
- Interview-to-decision ratios
- Department-wise hiring trends

**Built with:** Streamlit + read-only SQL views; export CSV/PDF

---

### 📋 8.8 Remaining Data Collection (Form-4)

**What it does:**
Post-selection but before joining: Candidate fills Form-4 during induction for remaining personal details capture.

---

## 9. Data Design

### 9.1 Overview

- **Style:** Normalized 3NF; UUID keys; UTC timestamps
- **Security:** PII encrypted at rest (DB/storage)
- **Identity:** Candidate identified by mobile + email
- **Application:** Per-position per candidate tracking

### 9.2 🗂 Core Entities

| Entity | Purpose | Key Columns (Sample) |
|--------|---------|----------------------|
| personal_details | Canonical candidate profile | candidate_id (PK), mobile_no (unique), email (unique), full_name, last_seen_at |
| candidate_applications | One per application/role | application_id (PK), candidate_id (FK), job_code, status, applied_at |
| addresses | Permanent/current addresses | address_id (PK), candidate_id (FK), type, city, state, pincode |
| qualifications | Degrees/certificates | qualification_id (PK), candidate_id (FK), degree, year |
| skills | Skill inventory | skill_id (PK), candidate_id (FK), skill_name, level |
| documents | Resume/KYC files | document_id (PK), candidate_id (FK), application_id (nullable FK), doc_type, signed_url |
| interviews | Scheduled interview slots | interview_id (PK), application_id (FK), slot_ts, mode, location |
| interview_evaluations | Panel feedback | evaluation_id (PK), interview_id (FK), criteria JSONB, overall_score, remarks |
| application_status_history | Immutable audit trail | id (PK), application_id (FK), from_status, to_status, changed_at, changed_by, note |

### 9.3 🔄 Data Flow Summary

| Form/Stage | Data Operations |
|------------|-----------------|
| Form-1 | Upsert personal_details, insert candidate_applications |
| Scheduling | Update candidate_applications.status; append to history |
| Form-2 | Insert addresses, qualifications, skills, documents |
| Interview (Form-3) | Insert interview_evaluations; update status |
| Decision | Set Selected/Rejected; append to application_status_history |

### 9.4 🔒 Integrity, Constraints & Security

#### Constraints
- Unique: personal_details.mobile_no, personal_details.email
- Foreign Keys: All children reference candidate_id or application_id
- Status Enum: applied, screened, interview_scheduled, interview_completed, selected, rejected, onboarded
- Check constraints on date fields and status transitions

#### Performance & Security
- **Indexes:**
  - idx_app_status (status, applied_at desc)
  - idx_eval_interview (interview_id)
  - idx_hist_app_time (application_id, changed_at desc)

- **RLS (Optional):** Candidate=self; Recruiter=branch/assigned; Interviewer=assigned

---

## 10. Deployment Architecture

### 10.1 🎯 Target Topology

```mermaid
graph TB
    USERS["🌐 Internet Users"]
    
    subgraph "Hostinger VPS (Ubuntu 22.04)"
        FIREWALL["🔥 Firewall<br/>Ports: 80, 443, 22"]
        
        subgraph "Docker Compose"
            CP["📱 Streamlit<br/>Candidate Portal<br/>Port 8501"]
            HP["📊 Streamlit<br/>HR Dashboard<br/>Port 8502"]
            API["⚡ FastAPI<br/>Port 8000"]
            N8N["🔄 n8n<br/>Internal"]
        end
    end
    
    SUPABASE["☁️ Supabase Cloud<br/>PostgreSQL<br/>Blob Storage<br/>Daily Backups"]
    
    USERS -->|HTTPS| FIREWALL
    FIREWALL -->|8501| CP
    FIREWALL -->|8502| HP
    FIREWALL -->|8000| API
    API -->|Private| SUPABASE
    CP -->|Private| SUPABASE
    HP -->|Private| SUPABASE
    API -->|Webhooks| N8N
    N8N -->|Send| USERS
    
    style USERS fill:#e3f2fd
    style FIREWALL fill:#ffcdd2
    style CP fill:#e3f2fd
    style HP fill:#f3e5f5
    style API fill:#fff3e0
    style N8N fill:#f1f8e9
    style SUPABASE fill:#c8e6c9
```

### 10.2 CI/CD Overview (Git/GitHub)

```mermaid
graph LR
    CODE["👨‍💻 Code Commit<br/>GitHub"]
    LINT["🔍 Lint"]
    TEST["✅ Test"]
    BUILD["🔨 Build Docker<br/>Image"]
    REGISTRY["📦 Push to<br/>Registry"]
    DEPLOY["🚀 Deploy<br/>docker-compose up"]
    MIGRATE["🗄️ Alembic<br/>Migrate"]
    SMOKE["🔥 Smoke Tests"]
    PROD["✨ Production"]
    
    CODE -->|Trigger| LINT
    LINT --> TEST
    TEST --> BUILD
    BUILD --> REGISTRY
    REGISTRY --> DEPLOY
    DEPLOY --> MIGRATE
    MIGRATE --> SMOKE
    SMOKE -->|Success| PROD
    
    style CODE fill:#e3f2fd
    style BUILD fill:#fff3e0
    style REGISTRY fill:#f3e5f5
    style DEPLOY fill:#c8e6c9
    style PROD fill:#81c784
```

### 10.3 💾 Backup & DR

| Component | Strategy | RPO/RTO |
|-----------|----------|---------|
| Database | Nightly full + 15-min WAL; retain 30 days | RPO: 15m / RTO: 4h |
| Storage | Versioning + lifecycle rules; cross-region optional | As per cloud provider SLA |
| Drill | Semi-annual restore rehearsal | Tested recovery procedures |

### 10.4 📊 Observability & Monitoring

#### Health Checks
- `/health` endpoint
- `/ready` endpoint
- Container logs with rotation

#### Alarms (Minimal)
- Latency spikes
- 5xx error rate
- Disk usage
- Failed backups

---

## 11. 🔐 Compliance & Data Retention

### 11.1 PII Handling (Personal Identifiable Information)

The system ensures **strict protection of all personally identifiable information (PII)** collected during recruitment and onboarding.

**PII includes:** candidate name, contact details, education, KYC, and interview evaluations.

| Compliance Area | Control Implemented |
|-----------------|---------------------|
| Data Encryption | AES-256 encryption for data at rest (DB + Blob storage), TLS 1.3 for in-transit |
| Access Control | Role-Based Access (RBAC): candidates access only their own data; HR/Admin restricted by scope |
| Masking & Logs | No plain-text PII in logs or exports; sensitive fields masked in error traces |
| Audit Trail | Immutable logs for all create/update/delete actions with timestamp, user, and IP |
| Consent Management | Candidates consent to data usage at Form-1 submission |
| Third-Party Data Sharing | Not applicable (no external integrations in Phase-1) |

### 11.2 📅 Data Retention Policy

| Data Category | Storage | Retention Duration | Notes |
|---------------|---------|-------------------|-------|
| Core Candidate Records | PostgreSQL | 3 years active + 2 years archive | Automatically anonymized after 5 years |
| Documents (KYC, Resume, Certificates) | Supabase Blob Storage | 7 years | Versioned and encrypted |
| Audit Logs & Application History | PostgreSQL (Partitioned Tables) | 10 years | Immutable, used for compliance review |
| Backups (DB + Blob) | Secure Cloud Bucket | 30 days rolling | Daily full backup + WAL every 15 min |

### 11.3 Compliance Alignment

```mermaid
graph LR
    GDPR["✅ GDPR<br/>Compliant<br/>Data Retention"]
    DPA["✅ Local DPA<br/>Encryption<br/>IN"]
    SOC2["✅ SOC2<br/>Audit Logs<br/>Backups"]
    
    GDPR -.->|Verified| CENTER{{"🔐 Compliance<br/>Framework"}}
    DPA -.->|Verified| CENTER
    SOC2 -.->|Verified| CENTER
    
    style GDPR fill:#c8e6c9
    style DPA fill:#c8e6c9
    style SOC2 fill:#c8e6c9
    style CENTER fill:#fff9c4
```

---

## 12. ⚠️ Risks & Mitigations

| Risk ID | Description | Impact | Likelihood | Owner | Mitigation / Action Plan |
|---------|-------------|--------|-----------|-------|--------------------------|
| R-01 | Data breach due to misconfiguration | High | Low | IT & Security | Enforce RBAC, use HTTPS, enable encryption & regular audits |
| R-02 | Server downtime or data loss | High | Medium | Infra/DevOps | Nightly DB backups, redundant Blob storage, DR drills every 6 months |
| R-03 | Unauthorized access by HR staff | Medium | Medium | HR Admin | Role segregation, MFA for HR Manager, activity logging |
| R-04 | Manual Excel uploads may cause data mismatch | Medium | High | HR Ops | Automate import validation and set verification rules |
| R-05 | Candidate duplicate records (mobile/email reuse) | Low | Medium | Backend Dev | Database-level unique constraints, pre-check logic |
| R-06 | Incomplete automation for background verification | Medium | High | HR Head | Manual verification to continue until Phase-2 automation |
| R-07 | Compliance violation (PII retention) | High | Low | Compliance Team | Apply auto-anonymization & annual retention review |
| R-08 | Network failure during onboarding sync | Medium | Low | IT Support | Local retries and queue-based message persistence |

---

## 13. Success Metrics & KPIs

### 13.1 System KPIs

| Parameter | Target | Monitoring Tool |
|-----------|--------|-----------------|
| API Response Time (p95) | < 2 seconds | FastAPI logs / metrics |
| System Uptime | ≥ 99.5% | Hostinger VPS monitoring |
| Data Accuracy | ≥ 99% | Periodic validation reports |
| Backup Success Rate | 100% daily | Backup logs |
| Security Incidents | 0 critical breaches | Audit reports |

### 13.2 💼 Business Outcome KPIs

```mermaid
graph TB
    KPI1["⏱️ Time-to-Hire<br/>↓ 30% reduction<br/>Target: 45 days"]
    KPI2["💰 Cost per Hire<br/>↓ 40% reduction<br/>HR workload"]
    KPI3["📊 Hire Quality<br/>↑ Interview-to-<br/>Offer ratio"]
    KPI4["📈 Candidate<br/>Satisfaction<br/>↑ Feedback scores"]
    KPI5["📉 Dropout Rate<br/>↓ 50% fewer<br/>incompletes"]
    
    KPI1 -.-> GOAL{{"🎯 Business<br/>Success"}}
    KPI2 -.-> GOAL
    KPI3 -.-> GOAL
    KPI4 -.-> GOAL
    KPI5 -.-> GOAL
    
    style KPI1 fill:#e3f2fd
    style KPI2 fill:#fff3e0
    style KPI3 fill:#c8e6c9
    style KPI4 fill:#f3e5f5
    style KPI5 fill:#fce4ec
    style GOAL fill:#fff9c4
```

### ✅ Transparency & Auditability
All actions logged & traceable for complete audit trail

### ✅ Compliance Readiness
100% adherence to internal audit checks

---

## 14. 🔄 HR Recruitment Process Flow

### 14.1 🛠️ Pre-requisite

#### 📱 Communication Setup
- Dedicated WhatsApp number
- WhatsApp message plan

#### 📊 Pre-Selection Phase
- Vacancy Identification via MIS data
- Job Post Creation and social media publishing

### 14.2 🧪 Selection Phase - Preliminary Selection

| Step | Action |
|------|--------|
| 1. CV Submission | Candidate sends CV to official WhatsApp number |
| 2. Form-1 Distribution | Automated WhatsApp message sends Form-1 (basic HR details) |
| 3. Form Submission | Candidate fills Form-1 → data stored → confirmation sent |
| 4. Evaluation | Selection criteria evaluated using predefined formula in database |
| 5. Interview Scheduling | HR manually inputs interview date, time, and location |
| 6. Interview Notification | Selected candidates receive WhatsApp + IVR call (R&D) + Chatbot support (R&D) |

### 14.3 🏢 Selection Phase - In-Office Interview

1. **QR Code Scan:** Candidate scans QR code to access test link
   - Previous stage candidates → Form-2
   - Walk-in candidates → Form-1 + Form-2

2. **Evaluation:** Candidate responses assessed

3. **Final Shortlisting:** HR refines selection and fills Form-3 for final shortlisted candidates

### 14.4 🔍 Post-Interview Processing

| Step | Details |
|------|---------|
| Background Verification | Done by HR (automation scope under R&D) |
| Status Update | Verification status updated in database |
| Salary Entry | Salary details entered for shortlisted candidates |
| Data Upload & Offer Letter | Data uploaded to Octane → offer letter generated |
| Communication | ✅ Selected: Offer letter + congratulations via email |
| | ❌ Rejected: Email with remarks |
| Post Data Entry | Candidate fills Form-4 during induction |

---

## Appendix: Complete System Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        CP["👥 Candidate Portal<br/>Streamlit 8501"]
        HP["📊 HR Dashboard<br/>Streamlit 8502"]
    end
    
    subgraph "Backend Layer"
        API["⚡ FastAPI<br/>Port 8000<br/>Business Logic"]
        AUTH["🔐 JWT Auth<br/>RBAC"]
    end
    
    subgraph "Data Layer"
        DB["🗄️ PostgreSQL<br/>Supabase"]
        CACHE["⚡ Redis Cache"]
    end
    
    subgraph "Storage Layer"
        BLOB["📦 Supabase<br/>Blob Storage"]
    end
    
    subgraph "Automation Layer"
        N8N["🔄 n8n<br/>Email, SMS<br/>WhatsApp"]
    end
    
    subgraph "Infrastructure"
        DOCKER["🐳 Docker Compose<br/>Hostinger VPS"]
        GIT["📘 GitHub<br/>CI/CD"]
    end
    
    CP -->|HTTPS| API
    HP -->|HTTPS| API
    API -->|Auth| AUTH
    API -->|Query/Cache| DB
    API -->|Cache| CACHE
    API -->|Files| BLOB
    API -->|Webhooks| N8N
    
    DOCKER -->|Manages| CP
    DOCKER -->|Manages| HP
    DOCKER -->|Manages| API
    DOCKER -->|Manages| N8N
    GIT -->|Deploys| DOCKER
    
    style CP fill:#e3f2fd
    style HP fill:#f3e5f5
    style API fill:#fff3e0
    style AUTH fill:#ffe0b2
    style DB fill:#c8e6c9
    style CACHE fill:#ffccbc
    style BLOB fill:#fce4ec
    style N8N fill:#f1f8e9
    style DOCKER fill:#eceff1
    style GIT fill:#ede7f6
```

---

**Document End**

**Generated:** 2025-12-30  
**Version:** 2.0  
**Format:** Markdown with Mermaid Diagrams (GitHub-Compatible)
