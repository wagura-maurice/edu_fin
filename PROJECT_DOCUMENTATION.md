# EduFin Kenya - Education Financing Platform

## Project Documentation

**Version:** 1.1  
**Date:** August 6, 2026  
**Status:** Planning Phase

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Role Taxonomy & Legal Framework](#2-role-taxonomy--legal-framework)
3. [Project Objectives](#3-project-objectives)
4. [System Architecture Overview](#4-system-architecture-overview)
5. [User Personas & Stakeholders](#5-user-personas--stakeholders)
6. [Functional Requirements](#6-functional-requirements)
7. [Technical Requirements](#7-technical-requirements)
8. [Data Models & Entities](#8-data-models--entities)
9. [Feature Specifications](#9-feature-specifications)
10. [Financing Products & Packages](#10-financing-products--packages)
11. [Collateral & Leverage Framework](#11-collateral--leverage-framework)
12. [Notification & Communication System](#12-notification--communication-system)
13. [Milestones & Deliverables](#13-milestones--deliverables)
14. [Task Breakdown](#14-task-breakdown)
15. [Risk Assessment](#15-risk-assessment)
16. [Compliance & Regulatory Considerations](#16-compliance--regulatory-considerations)
17. [Glossary](#17-glossary)

---

## 1. Executive Summary

EduFin Kenya is an education financing platform designed to address the challenge Kenyans face in paying school fees as lump-sum payments. The platform serves as a customer-facing onboarding system and client portal that integrates with a core banking system for actual financial operations.

### Core Value Proposition

- Enable individuals to finance education expenses through flexible loan products
- Provide installment-based payment options with flexible repayment schedules
- Support all levels of education from primary school to PhD programs
- Facilitate KYC-compliant onboarding for institutions, applicants, and beneficiaries

### System Boundaries

| In Scope (This Platform) | Out of Scope (Core Banking) |
|--------------------------|----------------------------|
| Customer onboarding & KYC | Loan disbursement |
| School & student registration | M-Pesa integration |
| Loan application processing | Bank API integrations |
| Client portal & statements | Payment processing |
| Collateral documentation | Credit scoring engine |
| Notification system | Financial reconciliation |

---

## 2. Role Taxonomy & Legal Framework

This section defines the professional terminology used throughout the platform to describe the functional and legal relationships between parties in an education financing arrangement. These terms replace traditional familial labels (e.g., "parent," "student") with role-based designations that accurately reflect legal obligations, financial responsibilities, and system permissions.

### 2.1 Core Role Definitions

| Role | Definition | Legal Standing |
|------|------------|----------------|
| **Primary Applicant** | The individual who initiates the loan application and undergoes full KYC verification | Contractual party to the loan agreement |
| **Principal Obligor** | The party bearing primary legal responsibility for loan repayment | Legally liable for debt obligations |
| **Co-Obligor** | An additional party sharing legal responsibility for loan repayment | Joint and several liability |
| **Education Beneficiary** | The individual whose education is being financed | Recipient of educational services funded by the loan |
| **Collateral Provider** | The party providing assets as security for the loan | Asset owner with lien obligations |
| **Account Holder** | The individual with primary access to the client portal and account management | System access and notification recipient |

### 2.2 Role Relationship Matrix

The following matrix illustrates how roles can be combined or assigned across different financing scenarios:

| Scenario | Primary Applicant | Principal Obligor | Education Beneficiary | Notes |
|----------|-------------------|-------------------|----------------------|-------|
| **Self-Financing Adult** | Individual | Same as Applicant | Same as Applicant | Single party holds all roles |
| **Third-Party Sponsorship (Minor)** | Sponsor | Sponsor | Minor Student | Sponsor retains full liability |
| **Third-Party Sponsorship (Adult)** | Sponsor | Sponsor + Beneficiary | Adult Student | Liability transfers or becomes joint upon beneficiary reaching 18 |
| **Guardian Financing** | Guardian | Guardian | Ward | Guardian liable until ward's majority |
| **Corporate Sponsorship** | Authorized Representative | Corporate Entity | Employee/Dependent | Corporate guarantee required |

### 2.3 Liability Transition Framework

A critical feature of education financing is the potential transfer of legal obligation when a minor beneficiary reaches the age of majority (18 years in Kenya).

#### 2.3.1 Liability States

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        LIABILITY TRANSITION MODEL                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  BENEFICIARY AGE < 18                    BENEFICIARY AGE ≥ 18               │
│  ┌─────────────────────┐                 ┌─────────────────────┐            │
│  │   SOLE OBLIGOR      │                 │   TRANSITION        │            │
│  │   STATE             │ ───────────────►│   OPTIONS           │            │
│  │                     │   Age of        │                     │            │
│  │ Primary Applicant   │   Majority      │ A) Sole Obligor     │            │
│  │ = Principal Obligor │   Trigger       │    (Original)       │            │
│  │                     │                 │                     │            │
│  │ Beneficiary has     │                 │ B) Joint Obligors   │            │
│  │ no legal liability  │                 │    (Shared)         │            │
│  │                     │                 │                     │            │
│  └─────────────────────┘                 │ C) Transferred      │            │
│                                          │    Obligor (New)    │            │
│                                          └─────────────────────┘            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### 2.3.2 Transition Options

| Option | Description | Legal Mechanism | System Action |
|--------|-------------|-----------------|---------------|
| **A) Retained Sole Obligor** | Original applicant remains solely liable | No change to agreement | Update beneficiary profile to reflect adult status |
| **B) Joint Obligor Addition** | Beneficiary becomes co-obligor alongside original applicant | Addendum to loan agreement; beneficiary signs | Create co-obligor record; enable beneficiary portal access |
| **C) Obligor Transfer** | Full liability transfers to beneficiary | Novation agreement; release of original obligor | Update principal obligor; transfer account ownership |

#### 2.3.3 Transition Triggers

The system shall monitor and initiate liability transition workflows based on:

- Beneficiary's 18th birthday (automatic notification)
- Loan renewal or restructuring events
- Explicit request by Primary Applicant
- Regulatory or compliance requirements

### 2.4 Role-Based Access Control (RBAC) Mapping

| System Role | Portal Access | Permissions |
|-------------|---------------|-------------|
| **Primary Applicant** | Full | Apply for loans, manage beneficiaries, view statements, make payments, update profile, manage collateral |
| **Principal Obligor** | Full | All Primary Applicant permissions (typically same person) |
| **Co-Obligor** | Standard | View statements, make payments, receive notifications |
| **Education Beneficiary (Minor)** | None | No direct system access |
| **Education Beneficiary (Adult)** | Limited | View own loan details, view statements, make payments |
| **Beneficiary with Elevated Access** | Standard | Granted by Primary Applicant; includes statement downloads, payment history |

### 2.5 Terminology Mapping (Legacy to Current)

To ensure clarity during transition and for stakeholder communication, the following mapping is provided:

| Legacy Term | Current Term | Context |
|-------------|--------------|---------|
| Parent | Primary Applicant / Principal Obligor | When applying for financing |
| Guardian | Primary Applicant / Principal Obligor | Legal guardian context |
| Student | Education Beneficiary | Recipient of funded education |
| Child | Dependent Beneficiary | Minor beneficiary |
| Adult Student | Self-Financing Applicant OR Adult Beneficiary | Context-dependent |
| Sponsor | Third-Party Applicant | Non-familial financing |
| Co-signer | Co-Obligor | Shared liability |
| Guarantor | Collateral Provider / Co-Obligor | Depending on guarantee type |

### 2.6 Entity Relationship Model (Role-Based)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│    ┌───────────────────┐         ┌───────────────────┐                     │
│    │   ACCOUNT         │         │   INSTITUTION     │                     │
│    │   HOLDER          │         │   (School)        │                     │
│    ├───────────────────┤         ├───────────────────┤                     │
│    │ account_id        │         │ institution_id    │                     │
│    │ kyc_status        │         │ name, level       │                     │
│    │ verification_date │         │ bank_details      │                     │
│    └────────┬──────────┘         └─────────┬─────────┘                     │
│             │                              │                                │
│             │ 1:N                          │ 1:N                           │
│             ▼                              ▼                                │
│    ┌───────────────────┐         ┌───────────────────┐                     │
│    │   LOAN            │◄────────│   EDUCATION       │                     │
│    │   FACILITY        │  1:1    │   BENEFICIARY     │                     │
│    ├───────────────────┤         ├───────────────────┤                     │
│    │ facility_id       │         │ beneficiary_id    │                     │
│    │ principal         │         │ date_of_birth     │                     │
│    │ status            │         │ enrollment_status │                     │
│    │ deadline          │         │ institution_id    │                     │
│    └────────┬──────────┘         │ fee_structure     │                     │
│             │                    └───────────────────┘                     │
│             │ N:M                                                          │
│             ▼                                                               │
│    ┌───────────────────┐                                                   │
│    │   OBLIGOR         │                                                   │
│    │   ASSIGNMENT      │                                                   │
│    ├───────────────────┤                                                   │
│    │ assignment_id     │                                                   │
│    │ account_holder_id │                                                   │
│    │ facility_id       │                                                   │
│    │ obligor_type      │  ← (PRIMARY | CO_OBLIGOR | TRANSFERRED)          │
│    │ liability_share   │  ← (percentage or JOINT_SEVERAL)                 │
│    │ effective_date    │                                                   │
│    │ termination_date  │                                                   │
│    └───────────────────┘                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.7 Use Case Scenarios

#### Scenario A: Self-Financing Professional

> *A 28-year-old professional enrolls in an MBA program and applies for financing.*

| Role | Assigned To |
|------|-------------|
| Primary Applicant | Self |
| Principal Obligor | Self |
| Education Beneficiary | Self |
| Collateral Provider | Self |
| Account Holder | Self |

**System Configuration:** Single-party account with unified roles.

#### Scenario B: Sponsored Minor (Primary School)

> *A guardian applies for financing for their 10-year-old ward's primary education.*

| Role | Assigned To |
|------|-------------|
| Primary Applicant | Guardian |
| Principal Obligor | Guardian |
| Education Beneficiary | Ward (Minor) |
| Collateral Provider | Guardian |
| Account Holder | Guardian |

**System Configuration:** Guardian-managed account; beneficiary has no portal access; no liability transition applicable during loan term.

#### Scenario C: University Student with Liability Transition

> *A parent applies for financing for their 17-year-old child's 4-year university degree.*

| Role | Initial Assignment | Post-18 Assignment |
|------|-------------------|-------------------|
| Primary Applicant | Parent | Parent |
| Principal Obligor | Parent | Parent + Student (Joint) |
| Education Beneficiary | Student | Student |
| Account Holder | Parent | Parent (Primary) + Student (Secondary) |

**System Configuration:** 
- Year 1: Parent-managed account
- Upon student turning 18: System triggers transition workflow
- Post-transition: Joint obligor status; student gains portal access with payment capabilities

#### Scenario D: Corporate Educational Sponsorship

> *An employer sponsors an employee's professional certification.*

| Role | Assigned To |
|------|-------------|
| Primary Applicant | HR Representative (Authorized Signatory) |
| Principal Obligor | Corporate Entity |
| Education Beneficiary | Employee |
| Collateral Provider | Corporate Entity (Corporate Guarantee) |
| Account Holder | HR Representative + Employee (View Only) |

**System Configuration:** Corporate account type; employee has limited view access; corporate guarantee documentation required.

### 2.8 Legal Considerations

#### 2.8.1 Age of Majority

Under Kenyan law (Age of Majority Act, Cap 33), an individual attains full legal capacity at 18 years. The system must:

- Track beneficiary date of birth
- Calculate age at loan origination and throughout loan lifecycle
- Prevent minors from being assigned as Principal Obligor
- Trigger transition workflows upon beneficiary reaching majority

#### 2.8.2 Contractual Capacity

| Party Type | Contractual Capacity | System Enforcement |
|------------|---------------------|-------------------|
| Adult (≥18) | Full capacity | Can be assigned any role |
| Minor (<18) | Limited capacity | Can only be Education Beneficiary |
| Corporate Entity | Via authorized representatives | Requires board resolution / authorization letter |

#### 2.8.3 Joint and Several Liability

When multiple obligors are assigned to a single loan facility:

- Default liability structure: **Joint and Several**
- Each obligor is individually liable for the full outstanding amount
- Payment by any obligor reduces the collective obligation
- System must maintain payment attribution for internal accounting

---

## 3. Project Objectives

### Primary Objectives

1. **Customer Onboarding System**
   - Implement comprehensive KYC workflows for parents/guardians
   - Enable school registration with fee structure documentation
   - Support student profile creation with institutional verification data

2. **Loan Application Platform**
   - Present education financing packages for all education levels
   - Capture collateral/leverage documentation
   - Generate loan paperwork compliant with Kenyan banking standards

3. **Client Portal**
   - Provide account management for parents with multiple children
   - Enable statement viewing and loan tracking
   - Support sub-accounts for adult students under guardian accounts

4. **Integration Readiness**
   - Prepare data structures for core banking system consumption
   - Ensure loan scoring prerequisites are captured
   - Maintain audit trails for compliance

### Success Metrics

| Metric | Target |
|--------|--------|
| Onboarding completion rate | > 80% |
| Document verification turnaround | < 48 hours |
| System uptime | 99.5% |
| Mobile responsiveness score | > 90 |

---

## 3. System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        EduFin Kenya Platform                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────┐  ┌──────────────────────────┐ │
│  │  WORDPRESS (edufin.co.ke)       │  │  LARAVEL (app.edufin.co.ke)│ │
│  │  Public Website                 │  │  Application              │ │
│  │  • Marketing, Blog, SEO         │  │  • Login (/login)         │ │
│  │  • Links to Laravel app         │  │  • Register (/register)   │ │
│  │    for login & registration     │  │  • Dashboard (/dashboard) │ │
│  └─────────────────────────────────┘  │  • Admin (/admin)         │ │
│                                       │  • REST API               │ │
│                                       │    (edufin.co.ke/api/v1)  │ │
│                                       └─────────────┬────────────┘ │
│                                                     │              │
│    ┌────────────────────────────────────────────────┼────────┐    │
│    │                     │                          │        │    │
│    ▼                     ▼                          ▼        ▼    │
│ ┌──────┐           ┌──────────┐         ┌───────────┐  ┌──────┐  │
│ │ KYC  │           │ Document │         │Notification│  │Core  │  │
│ │Engine│           │ Storage  │         │  Service   │  │Banking│  │
│ └──────┘           └──────────┘         └───────────┘  └──────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Description

| Component | Location | Purpose |
|-----------|----------|---------|
| Public Website | `edufin.co.ke` (WordPress) | Marketing, product information, blog |
| Login & Registration | `app.edufin.co.ke/login`, `/register` (Laravel) | User authentication and onboarding |
| Client Portal | `app.edufin.co.ke/dashboard` (Laravel/Livewire) | Account management, statements, loan tracking |
| Admin Panel | `app.edufin.co.ke/admin` (Laravel/Filament) | Staff administration, review workflows |
| API Layer | `edufin.co.ke/api/v1` (Laravel, path-based) | RESTful services for mobile app, webhooks, integrations |
| KYC Engine | Laravel application | Identity verification, document validation |
| Document Storage | Cloudflare R2 (`cdn.edufin.co.ke`) | Secure storage for legal documents, collateral proofs |
| Notification Service | Laravel application | Email and SMS alert management |
| Core Banking System | External | Financial operations, disbursements, payment processing |

> **Routing Notes:**
> - The REST API is served via path-based routing at `edufin.co.ke/api/v1/...` (Nginx proxies `/api/` to Laravel)
> - All users log in at `app.edufin.co.ke/login`; after authentication, role-based redirect sends clients to `/dashboard` and staff to `/admin`
> - User onboarding/registration occurs at `app.edufin.co.ke/register`
> - WordPress and Laravel are independent systems; WordPress links to the Laravel app via standard HTTP links

---

## 5. User Personas & Stakeholders

### Primary Users

#### 5.1 Primary Applicant (Third-Party Financing)

- **Role:** Account holder financing education for one or more beneficiaries
- **Typical Profile:** Parent, guardian, employer, or sponsor
- **Capabilities:**
  - Create and manage account with multiple Education Beneficiaries
  - Complete full KYC verification
  - Submit loan applications with collateral documentation
  - View statements and make payments
  - Manage beneficiary profiles and access permissions
  - Receive all account notifications

#### 5.2 Self-Financing Applicant

- **Role:** Individual financing their own education (unified Applicant/Beneficiary)
- **Typical Profile:** Adult learner, university student, professional development candidate
- **Capabilities:**
  - Full account holder privileges
  - Self-onboarding with personal KYC
  - Independent loan application and management
  - Direct statement access and payment control

#### 5.3 Adult Education Beneficiary (with Portal Access)

- **Role:** Beneficiary granted system access by Primary Applicant
- **Typical Profile:** University student (18+), adult dependent
- **Capabilities:**
  - View loan details for their education
  - Access statements and payment history
  - Make loan payments independently
  - Receive personal notifications
  - Cannot modify account settings or add beneficiaries

#### 5.4 Co-Obligor

- **Role:** Party sharing legal liability for loan repayment
- **Typical Profile:** Spouse, adult beneficiary (post-transition), business partner
- **Capabilities:**
  - View statements and loan status
  - Make payments on the facility
  - Receive payment reminders and overdue notices

### Secondary Stakeholders

| Stakeholder | Interest |
|-------------|----------|
| Educational Institutions | Fee structure validation, enrollment confirmation, payment receipt |
| Core Banking System | Loan data consumption, scoring inputs, disbursement triggers |
| Regulatory Bodies | KYC compliance, lending regulations, consumer protection |
| Internal Operations | Application review, document verification, liability management |
| Collateral Evaluators | Asset verification, valuation services |

---

## 6. Functional Requirements

### 6.1 Public Website

| ID | Requirement | Priority |
|----|-------------|----------|
| PW-01 | Display company information (About Us) | High |
| PW-02 | Present education financing products/packages | High |
| PW-03 | Show financing limits by education level | High |
| PW-04 | Provide loan calculator functionality | Medium |
| PW-05 | Link to application login/registration (app.edufin.co.ke/login, app.edufin.co.ke/register) | High |
| PW-06 | Mobile-responsive design | High |

### 6.2 Onboarding System

| ID | Requirement | Priority |
|----|-------------|----------|
| OB-01 | Primary Applicant KYC collection | High |
| OB-02 | Legal document upload (ID, Tax compliance) | High |
| OB-03 | Educational Institution registration with fee structure | High |
| OB-04 | Education Beneficiary profile creation | High |
| OB-05 | Collateral/leverage documentation | High |
| OB-06 | Multi-step application workflow | High |
| OB-07 | Document verification status tracking | Medium |
| OB-08 | Application save and resume | Medium |
| OB-09 | Self-financing applicant workflow (unified roles) | High |
| OB-10 | Beneficiary age verification and tracking | High |

### 6.3 Client Portal

| ID | Requirement | Priority |
|----|-------------|----------|
| CP-01 | Dashboard with loan facility overview | High |
| CP-02 | Statement generation and viewing | High |
| CP-03 | Payment history tracking | High |
| CP-04 | Multiple Education Beneficiary management | High |
| CP-05 | Beneficiary portal access provisioning | Medium |
| CP-06 | Document management | Medium |
| CP-07 | Profile settings and updates | Medium |
| CP-08 | Notification preferences | Low |
| CP-09 | Liability transition workflow interface | High |
| CP-10 | Co-Obligor management | Medium |

### 6.4 Administrative Functions

| ID | Requirement | Priority |
|----|-------------|----------|
| AD-01 | Application review dashboard | High |
| AD-02 | Document verification workflow | High |
| AD-03 | User and role management | High |
| AD-04 | Reporting and analytics | Medium |
| AD-05 | System configuration | Medium |
| AD-06 | Liability transition approval workflow | High |
| AD-07 | Obligor assignment management | Medium |

---

## 7. Technical Requirements

### 7.1 Technology Stack (Recommended)

| Layer | Technology Options |
|-------|-------------------|
| Frontend | React.js / Next.js |
| Backend | Node.js / Python (Django/FastAPI) |
| Database | PostgreSQL |
| File Storage | AWS S3 / Cloudflare R2 |
| Authentication | JWT + OAuth 2.0 |
| Email Service | SendGrid / AWS SES |
| SMS Gateway | Africa's Talking / Twilio |

### 7.2 Non-Functional Requirements

| Requirement | Specification |
|-------------|---------------|
| Performance | Page load < 3 seconds |
| Availability | 99.5% uptime SLA |
| Security | HTTPS, encryption at rest, OWASP compliance |
| Scalability | Support 10,000+ concurrent users |
| Data Retention | 7 years for financial records |
| Backup | Daily automated backups |

### 7.3 Integration Requirements

| Integration Point | Protocol | Purpose |
|-------------------|----------|---------|
| Core Banking System | REST API | Loan data export, status sync |
| NTSA (Future) | API | Vehicle logbook verification |
| Land Registry (Future) | API | Title deed verification |
| KRA (Future) | API | Tax compliance verification |

### 7.4 Security Requirements

- Multi-factor authentication (MFA) for sensitive operations
- Role-based access control (RBAC)
- Encrypted document storage
- Audit logging for all transactions
- Session management with timeout
- GDPR/Data Protection Act compliance

---

## 8. Data Models & Entities

### 8.1 Core Entities

```
┌─────────────────────┐       ┌─────────────────────┐
│   ACCOUNT HOLDER    │       │    INSTITUTION      │
├─────────────────────┤       ├─────────────────────┤
│ id                  │       │ id                  │
│ account_type        │       │ name                │
│ first_name          │       │ type                │
│ last_name           │       │ level               │
│ national_id         │       │ bank_account        │
│ kra_pin             │       │ location            │
│ email               │       │ contact_person      │
│ phone               │       │ verified            │
│ kyc_status          │       └──────────┬──────────┘
└──────────┬──────────┘                  │
           │                             │ 1:N
           │ 1:N                         ▼
           │              ┌─────────────────────────┐
           │              │  EDUCATION BENEFICIARY  │
           │              ├─────────────────────────┤
           │              │ id                      │
           │              │ first_name              │
           │              │ last_name               │
           │              │ date_of_birth           │
           │              │ institution_id          │
           │              │ admission_no            │
           │              │ is_self_financing       │
           │              │ portal_access_enabled   │
           │              └────────────┬────────────┘
           │                           │
           │ 1:N                       │ 1:1
           ▼                           ▼
┌─────────────────────┐     ┌─────────────────────┐
│   LOAN FACILITY     │◄────│   OBLIGOR           │
├─────────────────────┤     │   ASSIGNMENT        │
│ id                  │     ├─────────────────────┤
│ beneficiary_id      │     │ id                  │
│ principal           │     │ facility_id         │
│ interest_rate       │     │ account_holder_id   │
│ deadline            │     │ obligor_type        │
│ status              │     │ liability_share     │
│ collateral_id       │     │ effective_date      │
└─────────────────────┘     │ termination_date    │
                            └─────────────────────┘
```

### 8.2 Entity Specifications

#### Account Holder

The primary entity representing any individual who creates an account on the platform.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| account_type | Enum | INDIVIDUAL / CORPORATE |
| first_name | String | Legal first name |
| last_name | String | Legal surname |
| national_id | String | Kenyan National ID number |
| kra_pin | String | Kenya Revenue Authority PIN |
| email | String | Primary contact email |
| phone | String | Mobile number (M-Pesa registered) |
| address | Object | Physical address details |
| documents | Array | Uploaded KYC documents |
| kyc_status | Enum | PENDING / VERIFIED / REJECTED |
| kyc_verified_at | Timestamp | Verification completion date |
| created_at | Timestamp | Registration date |

#### Education Beneficiary

The individual whose education is being financed. May or may not have portal access.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| account_holder_id | UUID | Link to managing Account Holder |
| first_name | String | Beneficiary's first name |
| last_name | String | Beneficiary's surname |
| date_of_birth | Date | Used for age calculations and liability transitions |
| institution_id | UUID | Link to enrolled institution |
| admission_no | String | Institution admission/registration number |
| education_level | Enum | Current education level |
| program | String | Course/program name |
| fee_structure | Object | Term/semester fee breakdown |
| fee_document | File | Signed/stamped fee structure |
| is_self_financing | Boolean | True if beneficiary is also the Account Holder |
| portal_access_enabled | Boolean | Whether beneficiary has portal login |
| portal_user_id | UUID | Link to user account (if portal access enabled) |
| relationship_to_holder | String | Parent/Guardian/Self/Employer/Sponsor/Other |

#### Educational Institution

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| name | String | Official institution name |
| type | Enum | Public/Private |
| level | Enum | Primary/Secondary/Tertiary |
| category | String | University/College/Polytechnic/etc. |
| bank_name | String | Fee collection bank |
| bank_account | String | Fee collection account |
| paybill | String | M-Pesa paybill (if applicable) |
| location | Object | County, town, address |
| contact_person | String | Institution liaison |
| contact_phone | String | Institution contact number |
| verified | Boolean | Institution verification status |

#### Loan Facility

Represents a single financing arrangement for an Education Beneficiary.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| facility_reference | String | Human-readable reference number |
| beneficiary_id | UUID | Link to Education Beneficiary |
| principal | Decimal | Loan principal amount |
| interest_rate | Decimal | Annual interest rate |
| total_amount | Decimal | Principal + interest + fees |
| amount_paid | Decimal | Total payments received |
| outstanding_balance | Decimal | Remaining balance |
| initial_deposit | Decimal | Required minimum deposit |
| deadline | Date | Final repayment deadline |
| status | Enum | PENDING / ACTIVE / PAID / DEFAULTED / RESTRUCTURED |
| collateral_id | UUID | Link to Collateral record |
| created_at | Timestamp | Application date |
| activated_at | Timestamp | Loan activation date |

#### Obligor Assignment

Junction table managing the relationship between Account Holders and Loan Facilities, supporting multiple obligors per facility.

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| facility_id | UUID | Link to Loan Facility |
| account_holder_id | UUID | Link to Account Holder (obligor) |
| obligor_type | Enum | PRIMARY / CO_OBLIGOR / TRANSFERRED |
| liability_share | Enum | FULL / JOINT_SEVERAL / PERCENTAGE |
| liability_percentage | Decimal | If PERCENTAGE, the share (e.g., 50.00) |
| effective_date | Date | When this assignment became active |
| termination_date | Date | When this assignment ended (null if active) |
| termination_reason | String | Reason for termination (if applicable) |
| created_at | Timestamp | Record creation date |

#### Collateral/Leverage

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| account_holder_id | UUID | Owner reference (Collateral Provider) |
| type | Enum | Vehicle/Land/Property/Business/Investment/SACCO |
| description | String | Asset description |
| estimated_value | Decimal | Declared value |
| verified_value | Decimal | Verified value (if applicable) |
| documents | Array | Supporting documents |
| verification_status | Enum | PENDING / VERIFIED / REJECTED |
| verification_source | String | NTSA/Land Registry/Evaluator |
| lien_status | Enum | NONE / ACTIVE / RELEASED |
| lien_reference | String | External lien reference number |

---

## 9. Feature Specifications

### 9.1 KYC Onboarding Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Account   │───►│   Personal  │───►│   Document  │───►│   Review    │
│  Creation   │    │   Details   │    │   Upload    │    │  & Verify   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                │
┌─────────────┐    ┌─────────────┐    ┌─────────────┐           │
│   Active    │◄───│ Institution │◄───│ Beneficiary │◄──────────┘
│   Account   │    │   Details   │    │   Details   │
└─────────────┘    └─────────────┘    └─────────────┘
```

#### Required Documents (Primary Applicant / Account Holder)

| Document | Purpose | Validation |
|----------|---------|------------|
| National ID (Front & Back) | Identity verification | Manual review |
| KRA PIN Certificate | Tax compliance | KRA validation (future) |
| Passport Photo | Identity confirmation | Quality check |
| Proof of Address | Residence verification | Utility bill/bank statement |

#### Required Documents (Education Beneficiary)

| Document | Purpose | Validation |
|----------|---------|------------|
| Fee Structure | Payment breakdown | Institution stamp/signature required |
| Admission Letter | Enrollment confirmation | Institution letterhead |
| Student ID | Identity at institution | Photo matching |
| Birth Certificate (if minor) | Age verification | For liability transition tracking |

### 9.2 Loan Application Process

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Select    │───►│   Enter     │───►│   Upload    │───►│   Obligor   │
│   Package   │    │   Details   │    │  Collateral │    │  Assignment │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                               │
┌─────────────┐    ┌─────────────┐                             │
│   Loan      │◄───│   Review    │◄────────────────────────────┘
│   Active    │    │  & Approve  │
└─────────────┘    └─────────────┘
```

### 9.3 Payment Flexibility Model

Unlike traditional fixed-installment loans, EduFin implements a **flexible payment model**:

| Feature | Description |
|---------|-------------|
| Initial Deposit | Required minimum deposit to activate loan |
| Payment Deadline | Fixed end date for full repayment |
| Payment Flexibility | Any amount, any time before deadline |
| No Fixed Installments | No mandatory monthly/weekly payments |
| Early Payment | Allowed without penalties |
| Multi-Obligor Payments | Any assigned obligor can make payments |

### 9.4 Liability Transition Workflow

When an Education Beneficiary reaches the age of majority (18), the system initiates a transition workflow:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Age Trigger   │───►│   Notification  │───►│   Transition    │
│   (18th B'day)  │    │   to Parties    │    │   Selection     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                      │
        ┌─────────────────────────────────────────────┼─────────────────────────────────────────────┐
        │                                             │                                             │
        ▼                                             ▼                                             ▼
┌─────────────────┐                        ┌─────────────────┐                           ┌─────────────────┐
│   Option A:     │                        │   Option B:     │                           │   Option C:     │
│   No Change     │                        │   Add Co-Obligor│                           │   Full Transfer │
│   (Retain Sole) │                        │   (Joint)       │                           │   (Novation)    │
└─────────────────┘                        └─────────────────┘                           └─────────────────┘
        │                                             │                                             │
        ▼                                             ▼                                             ▼
┌─────────────────┐                        ┌─────────────────┐                           ┌─────────────────┐
│ Update Profile  │                        │ Beneficiary KYC │                           │ Beneficiary KYC │
│ (Adult Status)  │                        │ + Sign Addendum │                           │ + Novation Docs │
└─────────────────┘                        └─────────────────┘                           └─────────────────┘
```

---

## 10. Financing Products & Packages

### 10.1 Education Level Financing Limits

| Education Level | Maximum Financing (KES) | Typical Duration |
|-----------------|------------------------|------------------|
| Primary School | 200,000 | 1-3 years |
| Secondary School | 400,000 | 1-4 years |
| Certificate Programs | 300,000 | 6 months - 2 years |
| Diploma Programs | 500,000 | 1-3 years |
| Undergraduate Degree | 800,000 | 3-5 years |
| Postgraduate Diploma | 500,000 | 1-2 years |
| Master's Degree | 700,000 | 1-3 years |
| PhD Programs | 1,000,000 | 3-5 years |
| Technical/Polytechnic | 400,000 | 1-3 years |
| Professional Courses | 600,000 | Variable |

### 10.2 Fee Structure Types

| Education Level | Fee Period Type |
|-----------------|-----------------|
| Primary School | Term (3 per year) |
| Secondary School | Term (3 per year) |
| University | Semester (2 per year) |
| College | Semester/Trimester |
| Polytechnic | Trimester (3 per year) |
| Professional | Module-based |

### 10.3 Package Components

Each financing package includes:

- Principal amount (based on fee structure)
- Interest rate (determined by collateral and risk score)
- Processing fee
- Insurance levy (if applicable)
- Repayment deadline
- Minimum initial deposit requirement

---

## 11. Collateral & Leverage Framework

### 11.1 Accepted Collateral Types

| Collateral Type | Required Documents | Verification Method |
|-----------------|-------------------|---------------------|
| **Motor Vehicle** | Logbook, Insurance | NTSA verification |
| **Land** | Title Deed | Land Registry search |
| **Residential Property** | Title Deed, Valuation Report | Registered valuer |
| **Commercial Property** | Title Deed, Valuation Report | Registered valuer |
| **Business** | CR12, Financial Statements | Registrar of Companies |
| **Investments** | Portfolio Statement | Investment firm confirmation |
| **SACCO Shares** | Share Certificate, Statement | SACCO confirmation letter |
| **Fixed Deposits** | Bank Statement, Lien Letter | Bank confirmation |
| **Insurance Policies** | Policy Document | Insurance company confirmation |

### 11.2 Collateral Valuation Rules

| Collateral Type | Loan-to-Value (LTV) Ratio |
|-----------------|---------------------------|
| Motor Vehicle | Up to 60% of current value |
| Land (Urban) | Up to 70% of market value |
| Land (Rural) | Up to 50% of market value |
| Residential Property | Up to 70% of valuation |
| SACCO Shares | Up to 80% of share value |
| Fixed Deposits | Up to 90% of deposit value |

### 11.3 Verification Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Upload    │───►│   Initial   │───►│   External  │───►│   Final     │
│  Documents  │    │   Review    │    │ Verification│    │  Approval   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

---

## 12. Notification & Communication System

### 12.1 Communication Channels

| Channel | Priority | Use Cases |
|---------|----------|-----------|
| Email | Primary | All notifications, statements, documents |
| SMS | Secondary | Payment reminders, urgent alerts |
| In-App | Tertiary | Real-time updates, status changes |

### 12.2 Notification Types

| Notification | Channel | Trigger |
|--------------|---------|---------|
| Application Received | Email | On submission |
| Document Verification Status | Email | On status change |
| Loan Approval | Email + SMS | On approval |
| Payment Reminder | Email + SMS | 7, 3, 1 days before deadline |
| Payment Confirmation | Email | On payment receipt |
| Statement Available | Email | Monthly |
| Overdue Notice | Email + SMS | On deadline breach |
| Account Updates | Email | On profile changes |
| Liability Transition Notice | Email + SMS | On beneficiary turning 18 |
| Co-Obligor Addition | Email | On obligor assignment |
| Portal Access Granted | Email | On beneficiary access provisioning |

### 12.3 Notification Templates

All notifications must include:
- Clear subject line
- Loan/Account reference number
- Action required (if any)
- Contact information for support
- Unsubscribe option (where applicable)

---

## 13. Milestones & Deliverables

### Phase 1: Foundation (Weeks 1-4)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M1.1 Project Setup | Repository, CI/CD, environments | Week 1 |
| M1.2 Database Design | Schema, migrations, seed data | Week 1-2 |
| M1.3 Authentication | User registration, login, MFA | Week 2-3 |
| M1.4 Base UI Framework | Design system, layouts, navigation | Week 3-4 |

### Phase 2: Public Website (Weeks 5-7)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M2.1 Landing Page | Hero, features, CTA | Week 5 |
| M2.2 Product Pages | Package listings, details | Week 5-6 |
| M2.3 About & Contact | Company info, contact forms | Week 6 |
| M2.4 Loan Calculator | Interactive calculator tool | Week 7 |

### Phase 3: Onboarding System (Weeks 8-14)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M3.1 Primary Applicant KYC Flow | Multi-step form, document upload | Week 8-9 |
| M3.2 Institution Registration | Institution details, fee structure | Week 10-11 |
| M3.3 Education Beneficiary Profiles | Beneficiary details, linking | Week 11-12 |
| M3.4 Collateral Module | Asset documentation, upload | Week 12-13 |
| M3.5 Application Review | Admin verification workflow | Week 13-14 |

### Phase 4: Client Portal (Weeks 15-20)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M4.1 Dashboard | Overview, quick actions | Week 15-16 |
| M4.2 Loan Facility Management | Active facilities, details view | Week 16-17 |
| M4.3 Statements | Statement generation, download | Week 17-18 |
| M4.4 Beneficiary Management | Beneficiary profiles, access provisioning | Week 18-19 |
| M4.5 Notifications | Preferences, history | Week 19 |
| M4.6 Liability Transition | Transition workflows, obligor management | Week 19-20 |

### Phase 5: Integration & Testing (Weeks 21-24)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M5.1 API Documentation | OpenAPI specs, integration guide | Week 21 |
| M5.2 Core Banking Interface | Data export, webhook handlers | Week 21-22 |
| M5.3 UAT | User acceptance testing | Week 22-23 |
| M5.4 Performance Testing | Load testing, optimization | Week 23-24 |
| M5.5 Security Audit | Penetration testing, fixes | Week 24 |

### Phase 6: Launch (Weeks 25-26)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M6.1 Soft Launch | Beta users, monitoring | Week 25 |
| M6.2 Production Launch | Public release | Week 26 |
| M6.3 Documentation | User guides, admin manuals | Week 26 |

---

## 14. Task Breakdown

### 14.1 Backend Development

| Task ID | Task | Dependencies | Estimate |
|---------|------|--------------|----------|
| BE-01 | Project scaffolding & configuration | - | 2 days |
| BE-02 | Database schema implementation | BE-01 | 3 days |
| BE-03 | User authentication & authorization | BE-02 | 4 days |
| BE-04 | Account Holder CRUD APIs | BE-03 | 3 days |
| BE-05 | Institution management APIs | BE-03 | 2 days |
| BE-06 | Education Beneficiary management APIs | BE-04, BE-05 | 3 days |
| BE-07 | Document upload & storage | BE-03 | 3 days |
| BE-08 | Collateral management APIs | BE-04 | 4 days |
| BE-09 | Loan Facility APIs | BE-06, BE-08 | 5 days |
| BE-10 | Obligor Assignment APIs | BE-09 | 3 days |
| BE-11 | Statement generation service | BE-09 | 3 days |
| BE-12 | Notification service (Email) | BE-03 | 3 days |
| BE-13 | Notification service (SMS) | BE-12 | 2 days |
| BE-14 | Liability transition service | BE-10 | 4 days |
| BE-15 | Admin APIs | BE-03 | 4 days |
| BE-16 | Core banking integration layer | BE-09 | 5 days |
| BE-17 | API documentation | All | 2 days |

### 14.2 Frontend Development

| Task ID | Task | Dependencies | Estimate |
|---------|------|--------------|----------|
| FE-01 | Project setup & design system | - | 3 days |
| FE-02 | Authentication UI | FE-01, BE-03 | 3 days |
| FE-03 | Public website pages | FE-01 | 5 days |
| FE-04 | Loan calculator component | FE-03 | 2 days |
| FE-05 | Primary Applicant onboarding wizard | FE-02, BE-04 | 5 days |
| FE-06 | Document upload component | FE-05, BE-07 | 2 days |
| FE-07 | Institution registration form | FE-02, BE-05 | 3 days |
| FE-08 | Education Beneficiary profile forms | FE-07, BE-06 | 3 days |
| FE-09 | Collateral submission forms | FE-05, BE-08 | 4 days |
| FE-10 | Client dashboard | FE-02, BE-09 | 4 days |
| FE-11 | Loan facility details & statements | FE-10, BE-11 | 3 days |
| FE-12 | Beneficiary management UI | FE-10, BE-06 | 3 days |
| FE-13 | Liability transition UI | FE-12, BE-14 | 3 days |
| FE-14 | Notification center | FE-10, BE-12 | 2 days |
| FE-15 | Admin dashboard | FE-02, BE-15 | 5 days |
| FE-16 | Mobile responsiveness | All FE | 3 days |

### 14.3 DevOps & Infrastructure

| Task ID | Task | Dependencies | Estimate |
|---------|------|--------------|----------|
| DO-01 | Development environment setup | - | 1 day |
| DO-02 | CI/CD pipeline configuration | DO-01 | 2 days |
| DO-03 | Staging environment | DO-02 | 1 day |
| DO-04 | Production environment | DO-03 | 2 days |
| DO-05 | SSL/TLS configuration | DO-04 | 1 day |
| DO-06 | Monitoring & logging setup | DO-04 | 2 days |
| DO-07 | Backup automation | DO-04 | 1 day |
| DO-08 | CDN configuration | DO-04 | 1 day |

### 14.4 Quality Assurance

| Task ID | Task | Dependencies | Estimate |
|---------|------|--------------|----------|
| QA-01 | Test strategy & plan | - | 2 days |
| QA-02 | Unit test implementation | BE, FE tasks | Ongoing |
| QA-03 | Integration testing | BE-14 | 3 days |
| QA-04 | E2E test automation | FE tasks | 5 days |
| QA-05 | Performance testing | DO-04 | 3 days |
| QA-06 | Security testing | DO-04 | 4 days |
| QA-07 | UAT coordination | All | 5 days |

---

## 15. Risk Assessment

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| KYC verification delays | High | Medium | Implement manual review fallback |
| Document fraud | Medium | High | Multi-layer verification, audit trails |
| Core banking integration issues | Medium | High | Early API specification, mock services |
| Regulatory compliance gaps | Low | High | Legal review, compliance checklist |
| Data breach | Low | Critical | Security audit, encryption, access controls |
| System downtime | Low | High | Redundancy, monitoring, incident response |
| User adoption challenges | Medium | Medium | UX testing, onboarding tutorials |
| Scope creep | High | Medium | Change control process, MVP focus |
| Liability transition disputes | Medium | High | Clear documentation, legal review, audit trails |
| Minor beneficiary data handling | Medium | Medium | Age-appropriate data policies, parental consent |

---

## 16. Compliance & Regulatory Considerations

### 16.1 Kenyan Regulatory Framework

| Regulation | Applicability | Requirements |
|------------|---------------|--------------|
| Data Protection Act, 2019 | User data handling | Consent, data minimization, security |
| Central Bank of Kenya Guidelines | Lending operations | Via core banking partner |
| Consumer Protection Act | Customer relations | Fair terms, transparency |
| Anti-Money Laundering Act | KYC requirements | Identity verification, record keeping |
| Age of Majority Act (Cap 33) | Liability transitions | 18 years as age of contractual capacity |
| Children Act, 2001 | Minor beneficiary data | Parental consent, data protection for minors |

### 16.2 Compliance Checklist

- [ ] Privacy policy published
- [ ] Terms of service documented
- [ ] Cookie consent implementation
- [ ] Data retention policy defined
- [ ] User consent mechanisms
- [ ] Right to access/delete data
- [ ] Secure data transmission (HTTPS)
- [ ] Encrypted data storage
- [ ] Audit logging enabled
- [ ] Incident response plan
- [ ] Liability transition documentation templates
- [ ] Minor data handling procedures
- [ ] Parental consent mechanisms

---

## 17. Glossary

| Term | Definition |
|------|------------|
| **Account Holder** | Individual with a registered account on the platform; undergoes KYC verification |
| **Primary Applicant** | The party initiating a loan application; typically becomes Principal Obligor |
| **Principal Obligor** | Party bearing primary legal responsibility for loan repayment |
| **Co-Obligor** | Additional party sharing legal liability for a loan facility |
| **Education Beneficiary** | Individual whose education is being financed by the loan |
| **Self-Financing Applicant** | Individual who is both the Applicant and Beneficiary of the loan |
| **Collateral Provider** | Party providing assets as security; may differ from Principal Obligor |
| **Loan Facility** | A single financing arrangement for an Education Beneficiary |
| **Obligor Assignment** | Record linking an Account Holder to a Loan Facility with liability terms |
| **Liability Transition** | Process of transferring or sharing loan liability when beneficiary reaches 18 |
| **Novation** | Legal mechanism for transferring loan obligations to a new obligor |
| **Joint and Several Liability** | Each obligor is individually liable for the full loan amount |
| **KYC** | Know Your Customer - identity verification process |
| **Collateral** | Asset pledged as security for a loan |
| **Leverage** | Assets used to secure financing |
| **Principal** | Original loan amount before interest |
| **LTV** | Loan-to-Value ratio |
| **Core Banking System** | Backend financial system handling transactions |
| **Fee Structure** | Breakdown of educational fees by term/semester |
| **NTSA** | National Transport and Safety Authority |
| **KRA** | Kenya Revenue Authority |
| **SACCO** | Savings and Credit Cooperative Organization |
| **Paybill** | M-Pesa business payment number |
| **MFA** | Multi-Factor Authentication |
| **RBAC** | Role-Based Access Control |
| **UAT** | User Acceptance Testing |
| **Age of Majority** | Legal age of full contractual capacity (18 years in Kenya) |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Team | Initial documentation |
| 1.1 | 2026-08-06 | EduFin Team | Refined role taxonomy; replaced familial labels with functional/legal role-based terminology; added liability transition framework; updated data models for obligor assignments |
| 1.2 | 2026-08-06 | EduFin Team | Updated routing: API at edufin.co.ke/api/v1 (path-based); admin at app.edufin.co.ke/admin; login at app.edufin.co.ke/login with role-based redirect; onboarding at app.edufin.co.ke/register |

---

*This document serves as the master reference for the EduFin Kenya project. All stakeholders should refer to this document for project scope, requirements, and implementation guidelines.*
