# EduFin Kenya - Education Financing Platform

## Project Documentation

**Version:** 1.0  
**Date:** August 6, 2026  
**Status:** Planning Phase

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Project Objectives](#2-project-objectives)
3. [System Architecture Overview](#3-system-architecture-overview)
4. [User Personas & Stakeholders](#4-user-personas--stakeholders)
5. [Functional Requirements](#5-functional-requirements)
6. [Technical Requirements](#6-technical-requirements)
7. [Data Models & Entities](#7-data-models--entities)
8. [Feature Specifications](#8-feature-specifications)
9. [Financing Products & Packages](#9-financing-products--packages)
10. [Collateral & Leverage Framework](#10-collateral--leverage-framework)
11. [Notification & Communication System](#11-notification--communication-system)
12. [Milestones & Deliverables](#12-milestones--deliverables)
13. [Task Breakdown](#13-task-breakdown)
14. [Risk Assessment](#14-risk-assessment)
15. [Compliance & Regulatory Considerations](#15-compliance--regulatory-considerations)
16. [Glossary](#16-glossary)

---

## 1. Executive Summary

EduFin Kenya is an education financing platform designed to address the challenge Kenyan parents face in paying school fees as lump-sum payments. The platform serves as a customer-facing onboarding system and client portal that integrates with a core banking system for actual financial operations.

### Core Value Proposition

- Enable parents to finance education expenses through flexible loan products
- Provide installment-based payment options with flexible repayment schedules
- Support all levels of education from primary school to PhD programs
- Facilitate KYC-compliant onboarding for schools, parents, and students

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

## 2. Project Objectives

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
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │   Public    │  │  Onboarding │  │   Client    │                 │
│  │   Website   │  │    Portal   │  │   Portal    │                 │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘                 │
│         │                │                │                         │
│         └────────────────┼────────────────┘                         │
│                          │                                          │
│                    ┌─────┴─────┐                                    │
│                    │    API    │                                    │
│                    │   Layer   │                                    │
│                    └─────┬─────┘                                    │
│                          │                                          │
│    ┌─────────────────────┼─────────────────────┐                   │
│    │                     │                     │                    │
│    ▼                     ▼                     ▼                    │
│ ┌──────┐           ┌──────────┐         ┌───────────┐              │
│ │ KYC  │           │ Document │         │Notification│              │
│ │Engine│           │ Storage  │         │  Service   │              │
│ └──────┘           └──────────┘         └───────────┘              │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Core Banking      │
                    │   System (External) │
                    └─────────────────────┘
```

### Component Description

| Component | Purpose |
|-----------|---------|
| Public Website | Marketing, product information, package display |
| Onboarding Portal | KYC workflows, document upload, application forms |
| Client Portal | Account management, statements, loan tracking |
| API Layer | RESTful services, authentication, data validation |
| KYC Engine | Identity verification, document validation |
| Document Storage | Secure storage for legal documents, collateral proofs |
| Notification Service | Email and SMS alert management |

---

## 4. User Personas & Stakeholders

### Primary Users

#### 4.1 Parents/Guardians

- **Role:** Primary account holders and loan applicants
- **Capabilities:**
  - Create and manage family accounts
  - Onboard multiple children across different schools
  - Submit loan applications with collateral
  - View statements and make payments
  - Receive notifications on loan status

#### 4.2 Students (Adult)

- **Role:** Sub-account holders (university/tertiary level)
- **Capabilities:**
  - View loan statements under guardian account
  - Track payment schedules
  - Make loan payments independently
  - Receive personal notifications

#### 4.3 Individual Adult Learners

- **Role:** Self-sponsored students (e.g., university, professional courses)
- **Capabilities:**
  - Full account holder privileges
  - Self-onboarding with personal KYC
  - Independent loan application and management

### Secondary Stakeholders

| Stakeholder | Interest |
|-------------|----------|
| Schools/Institutions | Fee structure validation, payment confirmation |
| Core Banking System | Loan data consumption, scoring inputs |
| Regulatory Bodies | KYC compliance, lending regulations |
| Internal Operations | Application review, document verification |

---

## 5. Functional Requirements

### 5.1 Public Website

| ID | Requirement | Priority |
|----|-------------|----------|
| PW-01 | Display company information (About Us) | High |
| PW-02 | Present education financing products/packages | High |
| PW-03 | Show financing limits by education level | High |
| PW-04 | Provide loan calculator functionality | Medium |
| PW-05 | Enable user registration/login | High |
| PW-06 | Mobile-responsive design | High |

### 5.2 Onboarding System

| ID | Requirement | Priority |
|----|-------------|----------|
| OB-01 | Parent/Guardian KYC collection | High |
| OB-02 | Legal document upload (ID, Tax compliance) | High |
| OB-03 | School registration with fee structure | High |
| OB-04 | Student profile creation | High |
| OB-05 | Collateral/leverage documentation | High |
| OB-06 | Multi-step application workflow | High |
| OB-07 | Document verification status tracking | Medium |
| OB-08 | Application save and resume | Medium |

### 5.3 Client Portal

| ID | Requirement | Priority |
|----|-------------|----------|
| CP-01 | Dashboard with loan overview | High |
| CP-02 | Statement generation and viewing | High |
| CP-03 | Payment history tracking | High |
| CP-04 | Multiple children management | High |
| CP-05 | Sub-account creation for adult students | Medium |
| CP-06 | Document management | Medium |
| CP-07 | Profile settings and updates | Medium |
| CP-08 | Notification preferences | Low |

### 5.4 Administrative Functions

| ID | Requirement | Priority |
|----|-------------|----------|
| AD-01 | Application review dashboard | High |
| AD-02 | Document verification workflow | High |
| AD-03 | User management | High |
| AD-04 | Reporting and analytics | Medium |
| AD-05 | System configuration | Medium |

---

## 6. Technical Requirements

### 6.1 Technology Stack (Recommended)

| Layer | Technology Options |
|-------|-------------------|
| Frontend | React.js / Next.js |
| Backend | Node.js / Python (Django/FastAPI) |
| Database | PostgreSQL |
| File Storage | AWS S3 / Cloudflare R2 |
| Authentication | JWT + OAuth 2.0 |
| Email Service | SendGrid / AWS SES |
| SMS Gateway | Africa's Talking / Twilio |

### 6.2 Non-Functional Requirements

| Requirement | Specification |
|-------------|---------------|
| Performance | Page load < 3 seconds |
| Availability | 99.5% uptime SLA |
| Security | HTTPS, encryption at rest, OWASP compliance |
| Scalability | Support 10,000+ concurrent users |
| Data Retention | 7 years for financial records |
| Backup | Daily automated backups |

### 6.3 Integration Requirements

| Integration Point | Protocol | Purpose |
|-------------------|----------|---------|
| Core Banking System | REST API | Loan data export, status sync |
| NTSA (Future) | API | Vehicle logbook verification |
| Land Registry (Future) | API | Title deed verification |
| KRA (Future) | API | Tax compliance verification |

### 6.4 Security Requirements

- Multi-factor authentication (MFA) for sensitive operations
- Role-based access control (RBAC)
- Encrypted document storage
- Audit logging for all transactions
- Session management with timeout
- GDPR/Data Protection Act compliance

---

## 7. Data Models & Entities

### 7.1 Core Entities

```
┌─────────────────┐       ┌─────────────────┐
│     Parent      │       │     School      │
├─────────────────┤       ├─────────────────┤
│ id              │       │ id              │
│ first_name      │       │ name            │
│ last_name       │       │ type            │
│ national_id     │       │ level           │
│ kra_pin         │       │ bank_account    │
│ email           │       │ location        │
│ phone           │       │ contact_person  │
│ address         │       │ verified        │
└────────┬────────┘       └────────┬────────┘
         │                         │
         │    ┌─────────────────┐  │
         └───►│    Student      │◄─┘
              ├─────────────────┤
              │ id              │
              │ first_name      │
              │ last_name       │
              │ school_id       │
              │ parent_id       │
              │ admission_no    │
              │ education_level │
              │ fee_structure   │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              │  Loan Account   │
              ├─────────────────┤
              │ id              │
              │ student_id      │
              │ principal       │
              │ interest_rate   │
              │ duration        │
              │ deadline        │
              │ status          │
              │ collateral_id   │
              └─────────────────┘
```

### 7.2 Entity Specifications

#### Parent/Guardian

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| first_name | String | Legal first name |
| last_name | String | Legal surname |
| national_id | String | Kenyan National ID number |
| kra_pin | String | Kenya Revenue Authority PIN |
| email | String | Primary contact email |
| phone | String | Mobile number (M-Pesa registered) |
| address | Object | Physical address details |
| documents | Array | Uploaded KYC documents |
| created_at | Timestamp | Registration date |
| verified | Boolean | KYC verification status |

#### School/Institution

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
| contact_person | String | School liaison |
| contact_phone | String | School contact number |

#### Student

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| first_name | String | Student's first name |
| last_name | String | Student's surname |
| parent_id | UUID | Link to parent/guardian |
| school_id | UUID | Link to enrolled school |
| admission_no | String | School admission/registration number |
| education_level | Enum | Current education level |
| program | String | Course/program name |
| fee_structure | Object | Term/semester fee breakdown |
| fee_document | File | Signed/stamped fee structure |

#### Collateral/Leverage

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary identifier |
| parent_id | UUID | Owner reference |
| type | Enum | Vehicle/Land/Property/Business/Investment/SACCO |
| description | String | Asset description |
| estimated_value | Decimal | Declared value |
| verified_value | Decimal | Verified value (if applicable) |
| documents | Array | Supporting documents |
| verification_status | Enum | Pending/Verified/Rejected |
| verification_source | String | NTSA/Land Registry/Evaluator |

---

## 8. Feature Specifications

### 8.1 KYC Onboarding Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Account   │───►│   Personal  │───►│   Document  │───►│   Review    │
│  Creation   │    │   Details   │    │   Upload    │    │  & Verify   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                                │
┌─────────────┐    ┌─────────────┐    ┌─────────────┐           │
│   Active    │◄───│   School    │◄───│   Student   │◄──────────┘
│   Account   │    │   Details   │    │   Details   │
└─────────────┘    └─────────────┘    └─────────────┘
```

#### Required Documents (Parent/Guardian)

| Document | Purpose | Validation |
|----------|---------|------------|
| National ID (Front & Back) | Identity verification | Manual review |
| KRA PIN Certificate | Tax compliance | KRA validation (future) |
| Passport Photo | Identity confirmation | Quality check |
| Proof of Address | Residence verification | Utility bill/bank statement |

#### Required Documents (Student)

| Document | Purpose | Validation |
|----------|---------|------------|
| Fee Structure | Payment breakdown | School stamp/signature required |
| Admission Letter | Enrollment confirmation | School letterhead |
| Student ID | Identity at institution | Photo matching |

### 8.2 Loan Application Process

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Select    │───►│   Enter     │───►│   Upload    │
│   Package   │    │   Details   │    │  Collateral │
└─────────────┘    └─────────────┘    └─────────────┘
                                            │
┌─────────────┐    ┌─────────────┐          │
│   Loan      │◄───│   Review    │◄─────────┘
│   Active    │    │  & Approve  │
└─────────────┘    └─────────────┘
```

### 8.3 Payment Flexibility Model

Unlike traditional fixed-installment loans, EduFin implements a **flexible payment model**:

| Feature | Description |
|---------|-------------|
| Initial Deposit | Required minimum deposit to activate loan |
| Payment Deadline | Fixed end date for full repayment |
| Payment Flexibility | Any amount, any time before deadline |
| No Fixed Installments | No mandatory monthly/weekly payments |
| Early Payment | Allowed without penalties |

---

## 9. Financing Products & Packages

### 9.1 Education Level Financing Limits

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

### 9.2 Fee Structure Types

| Education Level | Fee Period Type |
|-----------------|-----------------|
| Primary School | Term (3 per year) |
| Secondary School | Term (3 per year) |
| University | Semester (2 per year) |
| College | Semester/Trimester |
| Polytechnic | Trimester (3 per year) |
| Professional | Module-based |

### 9.3 Package Components

Each financing package includes:

- Principal amount (based on fee structure)
- Interest rate (determined by collateral and risk score)
- Processing fee
- Insurance levy (if applicable)
- Repayment deadline
- Minimum initial deposit requirement

---

## 10. Collateral & Leverage Framework

### 10.1 Accepted Collateral Types

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

### 10.2 Collateral Valuation Rules

| Collateral Type | Loan-to-Value (LTV) Ratio |
|-----------------|---------------------------|
| Motor Vehicle | Up to 60% of current value |
| Land (Urban) | Up to 70% of market value |
| Land (Rural) | Up to 50% of market value |
| Residential Property | Up to 70% of valuation |
| SACCO Shares | Up to 80% of share value |
| Fixed Deposits | Up to 90% of deposit value |

### 10.3 Verification Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Upload    │───►│   Initial   │───►│   External  │───►│   Final     │
│  Documents  │    │   Review    │    │ Verification│    │  Approval   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

---

## 11. Notification & Communication System

### 11.1 Communication Channels

| Channel | Priority | Use Cases |
|---------|----------|-----------|
| Email | Primary | All notifications, statements, documents |
| SMS | Secondary | Payment reminders, urgent alerts |
| In-App | Tertiary | Real-time updates, status changes |

### 11.2 Notification Types

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

### 11.3 Notification Templates

All notifications must include:
- Clear subject line
- Loan/Account reference number
- Action required (if any)
- Contact information for support
- Unsubscribe option (where applicable)

---

## 12. Milestones & Deliverables

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
| M3.1 Parent KYC Flow | Multi-step form, document upload | Week 8-9 |
| M3.2 School Registration | School details, fee structure | Week 10-11 |
| M3.3 Student Profiles | Student details, linking | Week 11-12 |
| M3.4 Collateral Module | Asset documentation, upload | Week 12-13 |
| M3.5 Application Review | Admin verification workflow | Week 13-14 |

### Phase 4: Client Portal (Weeks 15-20)

| Milestone | Deliverables | Duration |
|-----------|--------------|----------|
| M4.1 Dashboard | Overview, quick actions | Week 15-16 |
| M4.2 Loan Management | Active loans, details view | Week 16-17 |
| M4.3 Statements | Statement generation, download | Week 17-18 |
| M4.4 Family Management | Children profiles, sub-accounts | Week 18-19 |
| M4.5 Notifications | Preferences, history | Week 19-20 |

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

## 13. Task Breakdown

### 13.1 Backend Development

| Task ID | Task | Dependencies | Estimate |
|---------|------|--------------|----------|
| BE-01 | Project scaffolding & configuration | - | 2 days |
| BE-02 | Database schema implementation | BE-01 | 3 days |
| BE-03 | User authentication & authorization | BE-02 | 4 days |
| BE-04 | Parent/Guardian CRUD APIs | BE-03 | 3 days |
| BE-05 | School management APIs | BE-03 | 2 days |
| BE-06 | Student management APIs | BE-04, BE-05 | 3 days |
| BE-07 | Document upload & storage | BE-03 | 3 days |
| BE-08 | Collateral management APIs | BE-04 | 4 days |
| BE-09 | Loan application APIs | BE-06, BE-08 | 5 days |
| BE-10 | Statement generation service | BE-09 | 3 days |
| BE-11 | Notification service (Email) | BE-03 | 3 days |
| BE-12 | Notification service (SMS) | BE-11 | 2 days |
| BE-13 | Admin APIs | BE-03 | 4 days |
| BE-14 | Core banking integration layer | BE-09 | 5 days |
| BE-15 | API documentation | All | 2 days |

### 13.2 Frontend Development

| Task ID | Task | Dependencies | Estimate |
|---------|------|--------------|----------|
| FE-01 | Project setup & design system | - | 3 days |
| FE-02 | Authentication UI | FE-01, BE-03 | 3 days |
| FE-03 | Public website pages | FE-01 | 5 days |
| FE-04 | Loan calculator component | FE-03 | 2 days |
| FE-05 | Parent onboarding wizard | FE-02, BE-04 | 5 days |
| FE-06 | Document upload component | FE-05, BE-07 | 2 days |
| FE-07 | School registration form | FE-02, BE-05 | 3 days |
| FE-08 | Student profile forms | FE-07, BE-06 | 3 days |
| FE-09 | Collateral submission forms | FE-05, BE-08 | 4 days |
| FE-10 | Client dashboard | FE-02, BE-09 | 4 days |
| FE-11 | Loan details & statements | FE-10, BE-10 | 3 days |
| FE-12 | Family management UI | FE-10, BE-06 | 3 days |
| FE-13 | Notification center | FE-10, BE-11 | 2 days |
| FE-14 | Admin dashboard | FE-02, BE-13 | 5 days |
| FE-15 | Mobile responsiveness | All FE | 3 days |

### 13.3 DevOps & Infrastructure

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

### 13.4 Quality Assurance

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

## 14. Risk Assessment

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

---

## 15. Compliance & Regulatory Considerations

### 15.1 Kenyan Regulatory Framework

| Regulation | Applicability | Requirements |
|------------|---------------|--------------|
| Data Protection Act, 2019 | User data handling | Consent, data minimization, security |
| Central Bank of Kenya Guidelines | Lending operations | Via core banking partner |
| Consumer Protection Act | Customer relations | Fair terms, transparency |
| Anti-Money Laundering Act | KYC requirements | Identity verification, record keeping |

### 15.2 Compliance Checklist

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

---

## 16. Glossary

| Term | Definition |
|------|------------|
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

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Team | Initial documentation |

---

*This document serves as the master reference for the EduFin Kenya project. All stakeholders should refer to this document for project scope, requirements, and implementation guidelines.*
