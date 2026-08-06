# EduFin - Project Documentation

## Global Project Overview

**Version:** 1.0  
**Last Updated:** August 6, 2026  
**Classification:** Project Documentation

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Business Context](#2-business-context)
3. [System Overview](#3-system-overview)
4. [Platform Architecture](#4-platform-architecture)
5. [User Personas & Roles](#5-user-personas--roles)
6. [Core Business Processes](#6-core-business-processes)
7. [Data Architecture](#7-data-architecture)
8. [Integration Landscape](#8-integration-landscape)
9. [Security & Compliance](#9-security--compliance)
10. [Project Roadmap](#10-project-roadmap)

---

## 1. Executive Summary

### 1.1 Project Vision

EduFin is a digital education financing platform designed to make quality education accessible to Kenyan families by providing flexible, transparent, and affordable financing solutions for educational expenses across all levels—from primary school to postgraduate studies.

### 1.2 Business Objectives

| Objective | Description | Success Metric |
|-----------|-------------|----------------|
| **Financial Inclusion** | Enable families to access education financing | 10,000 active clients in Year 1 |
| **Digital Transformation** | Paperless loan application process | 95% digital applications |
| **Operational Efficiency** | Streamlined KYC and approval workflows | 48-hour approval turnaround |
| **Customer Experience** | Self-service portal for clients | 80% self-service adoption |
| **Risk Management** | Robust collateral and credit assessment | NPL ratio < 5% |

### 1.3 Platform Summary

EduFin operates as a **dual-platform ecosystem**:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           EDUFIN PLATFORM ECOSYSTEM                             │
├─────────────────────────────────┬───────────────────────────────────────────────┤
│                                 │                                               │
│   WORDPRESS                     │   LARAVEL                                     │
│   Company Landing Page          │   Core Application Engine                     │
│                                 │                                               │
│   ┌─────────────────────────┐   │   ┌───────────────────────────────────────┐ │
│   │                         │   │   │                                       │ │
│   │  • Brand Presence       │   │   │  INTERNAL OPERATIONS                  │ │
│   │  • Marketing Content    │   │   │  • Staff Administration               │ │
│   │  • SEO & Discovery      │   │   │  • Loan Processing                    │ │
│   │  • Company Information  │   │   │  • KYC Verification                   │ │
│   │  • Blog & News          │   │   │  • Reporting & Analytics              │ │
│   │  • Contact & Support    │   │   │                                       │ │
│   │                         │   │   │  EXTERNAL CLIENT PORTAL               │ │
│   │  Managed by:            │   │   │  • Client Dashboard                   │ │
│   │  Secretarial Staff      │   │   │  • Loan Applications                  │ │
│   │                         │   │   │  • Document Management                │ │
│   │  Decoupled from         │   │   │  • Payment Tracking                   │ │
│   │  core application       │   │   │                                       │ │
│   │                         │   │   │  API LAYER                            │ │
│   └─────────────────────────┘   │   │  • Mobile App Integration             │ │
│                                 │   │  • Third-Party Integrations           │ │
│                                 │   │                                       │ │
│                                 │   │  BANKING INTEGRATION                  │ │
│                                 │   │  • Core Banking System                │ │
│                                 │   │  • Payment Processing                 │ │
│                                 │   │  • Reconciliation                     │ │
│                                 │   │                                       │ │
│                                 │   └───────────────────────────────────────┘ │
│                                 │                                               │
└─────────────────────────────────┴───────────────────────────────────────────────┘
```

---

## 2. Business Context

### 2.1 Problem Statement

Kenya faces significant challenges in education financing:

- **Affordability Gap:** Rising education costs outpace household income growth
- **Limited Access:** Traditional banks require extensive collateral and have lengthy approval processes
- **Timing Mismatch:** School fees are due at specific times, often misaligned with income cycles
- **Information Asymmetry:** Families lack visibility into financing options

### 2.2 Solution Overview

EduFin addresses these challenges through:

| Challenge | EduFin Solution |
|-----------|-----------------|
| Affordability | Flexible repayment terms aligned with income cycles |
| Access | Simplified KYC with alternative collateral options |
| Timing | Quick disbursement directly to educational institutions |
| Information | Transparent pricing and self-service portal |

### 2.3 Target Market

**Primary Market:**
- Middle-income Kenyan families (KES 50,000 - 300,000 monthly household income)
- Parents/guardians financing children's education
- Working professionals pursuing further education

**Secondary Market:**
- Corporate sponsors financing employee education
- NGOs and foundations supporting beneficiaries
- Educational institutions seeking financing partnerships

### 2.4 Regulatory Environment

| Regulation | Applicability | Compliance Approach |
|------------|---------------|---------------------|
| Data Protection Act, 2019 | PII handling | Encryption, consent management, data minimization |
| CBK Prudential Guidelines | Lending operations | Via Core Banking System partnership |
| Consumer Protection Act | Client interactions | Transparent pricing, fair collection practices |
| AML/CFT Regulations | Customer onboarding | KYC verification, transaction monitoring |

---

## 3. System Overview

### 3.1 Domain Architecture

| Domain | Platform | Purpose | Audience |
|--------|----------|---------|----------|
| `edufin.co.ke` | WordPress | Company landing page, marketing | Public |
| `app.edufin.co.ke` | Laravel | Client & staff portal | Authenticated users |
| `api.edufin.co.ke` | Laravel | REST API for mobile & integrations | Applications |
| `admin.edufin.co.ke` | Laravel/Filament | Internal administration | Staff only |

### 3.2 Technology Stack

| Layer | WordPress | Laravel |
|-------|-----------|---------|
| **Purpose** | Landing Page | Core Application |
| **Runtime** | PHP 8.2 | PHP 8.3 |
| **Framework** | WordPress 6.x | Laravel 11.x + Livewire 3.x |
| **Database** | MySQL 8.0 | PostgreSQL 16 |
| **Cache** | Redis | Redis |
| **Admin UI** | WordPress Admin | Filament 3.x |
| **API** | N/A | REST + Sanctum |

### 3.3 Architectural Principles

1. **Separation of Concerns**
   - WordPress handles ONLY marketing content
   - Laravel handles ALL business logic and data

2. **Security by Design**
   - No PII in WordPress database
   - Financial operations isolated in Laravel
   - Core banking integration via secure APIs

3. **API-First Architecture**
   - All business logic exposed via versioned APIs
   - Enables mobile app and future integrations

4. **Scalability**
   - Stateless application design
   - Horizontal scaling capability
   - Managed database services

---

## 4. Platform Architecture

### 4.1 WordPress - Company Landing Page

**Role:** Public-facing brand presence and marketing content management.

**Characteristics:**
- Standalone component, decoupled from core application
- Managed internally by company secretarial staff
- No access to business data or client information
- Communicates with Laravel only for SSO and public data (rates, packages)

**Content Managed:**
- Company profile and about information
- Product/service descriptions
- Blog articles and news
- FAQs and help content
- Contact information
- Legal documents (Terms, Privacy Policy)

**Administrative Access:**
- Secretarial staff: Content editing, media management
- IT Admin: Plugin updates, security patches

> **See:** [WordPress Architecture](./architecture/wordpress/README.md)

### 4.2 Laravel - Core Application Engine

**Role:** Central engine powering all business operations, client interactions, and external integrations.

**Capabilities:**

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        LARAVEL CORE APPLICATION ENGINE                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     INTERNAL OPERATIONS                                  │  │
│  │                                                                          │  │
│  │  Staff Portal (Filament)           Business Services                    │  │
│  │  ├── User Management               ├── KYC Verification                 │  │
│  │  ├── Role & Permission Mgmt        ├── Loan Processing                  │  │
│  │  ├── Client Account Review         ├── Collateral Management            │  │
│  │  ├── Loan Application Review       ├── Payment Reconciliation           │  │
│  │  ├── KYC Verification Queue        ├── Statement Generation             │  │
│  │  ├── Collateral Approval           ├── Notification Dispatch            │  │
│  │  ├── Disbursement Approval         └── Audit Logging                    │  │
│  │  ├── Reports & Analytics                                                │  │
│  │  └── System Configuration                                               │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     EXTERNAL CLIENT PORTAL                               │  │
│  │                                                                          │  │
│  │  Client Dashboard (Livewire)       Self-Service Features                │  │
│  │  ├── Account Overview              ├── Profile Management               │  │
│  │  ├── Active Loans                  ├── KYC Document Upload              │  │
│  │  ├── Payment Schedule              ├── Loan Application                 │  │
│  │  ├── Transaction History           ├── Beneficiary Management           │  │
│  │  ├── Statements & Documents        ├── Collateral Registration          │  │
│  │  └── Notifications                 └── Support Requests                 │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     API LAYER                                            │  │
│  │                                                                          │  │
│  │  REST API (api.edufin.co.ke)       Consumers                            │  │
│  │  ├── Authentication (JWT)          ├── Flutter Mobile App (iOS/Android) │  │
│  │  ├── Client Endpoints              ├── WordPress (SSO, Public Data)     │  │
│  │  ├── Loan Endpoints                ├── Future Partner Integrations      │  │
│  │  ├── Document Endpoints            └── Internal Microservices           │  │
│  │  ├── Notification Endpoints                                             │  │
│  │  └── Webhook Receivers                                                  │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     BANKING INTEGRATION LAYER                            │  │
│  │                                                                          │  │
│  │  Core Banking Interface            External Services                    │  │
│  │  ├── Loan Account Creation         ├── M-Pesa (Payments)                │  │
│  │  ├── Disbursement Requests         ├── Africa's Talking (SMS)           │  │
│  │  ├── Payment Processing            ├── SendGrid (Email)                 │  │
│  │  ├── Balance Inquiries             ├── NTSA (Vehicle Verification)      │  │
│  │  ├── Statement Retrieval           └── Land Registry (Title Search)     │  │
│  │  ├── Reconciliation Jobs                                                │  │
│  │  └── Webhook Processing                                                 │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

> **See:** [Laravel Architecture](./architecture/laravel/README.md)

---

## 5. User Personas & Roles

### 5.1 Role Taxonomy

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              USER ROLE HIERARCHY                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  INTERNAL ROLES (Staff)                                                        │
│  ══════════════════════                                                        │
│                                                                                 │
│  ┌─────────────────┐                                                           │
│  │ System          │  Full system access, configuration, user management      │
│  │ Administrator   │  Platform: Laravel (Filament)                            │
│  └────────┬────────┘                                                           │
│           │                                                                     │
│  ┌────────┴────────┐                                                           │
│  │ Operations      │  Loan approvals, KYC verification, reporting             │
│  │ Manager         │  Platform: Laravel (Filament)                            │
│  └────────┬────────┘                                                           │
│           │                                                                     │
│  ┌────────┴────────┐                                                           │
│  │ Loan            │  Process applications, document review                   │
│  │ Officer         │  Platform: Laravel (Filament)                            │
│  └─────────────────┘                                                           │
│                                                                                 │
│  ┌─────────────────┐                                                           │
│  │ Secretarial     │  Content management, marketing updates                   │
│  │ Staff           │  Platform: WordPress Admin (ONLY)                        │
│  └─────────────────┘                                                           │
│                                                                                 │
│  EXTERNAL ROLES (Clients)                                                      │
│  ════════════════════════                                                      │
│                                                                                 │
│  ┌─────────────────┐                                                           │
│  │ Primary         │  Account holder, loan applicant, full portal access      │
│  │ Applicant       │  Platform: Laravel Portal + Mobile App                   │
│  └────────┬────────┘                                                           │
│           │                                                                     │
│  ┌────────┴────────┐                                                           │
│  │ Education       │  View-only access to related loan information            │
│  │ Beneficiary     │  Platform: Laravel Portal + Mobile App (Limited)         │
│  └─────────────────┘                                                           │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Role-Platform Matrix

| Role | WordPress | Laravel Portal | Laravel Admin | Mobile App |
|------|:---------:|:--------------:|:-------------:|:----------:|
| System Administrator | - | - | Full Access | - |
| Operations Manager | - | - | Operational | - |
| Loan Officer | - | - | Limited | - |
| Secretarial Staff | Editor | - | - | - |
| Primary Applicant | - | Full Access | - | Full Access |
| Education Beneficiary | - | Limited | - | Limited |

### 5.3 Legal Role Framework

For loan facilities, EduFin implements a comprehensive legal role framework:

| Role | Definition | Legal Responsibility |
|------|------------|---------------------|
| **Primary Applicant** | Person initiating the loan application | Initial contact, application submission |
| **Principal Obligor** | Primary party legally bound to repay | Full repayment liability |
| **Co-Obligor** | Secondary party sharing repayment obligation | Joint and several liability |
| **Education Beneficiary** | Student receiving educational benefit | No direct financial liability (unless transitioned) |
| **Collateral Provider** | Party providing security for the loan | Asset at risk upon default |
| **Account Holder** | Party with portal access and account management | Account management, not necessarily liable |

> **See:** [PROJECT_DOCUMENTATION.md](../PROJECT_DOCUMENTATION.md) for complete role taxonomy

---

## 6. Core Business Processes

### 6.1 Client Onboarding

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           CLIENT ONBOARDING PROCESS                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐│
│  │          │    │          │    │          │    │          │    │          ││
│  │ Register │───►│  Verify  │───►│  Submit  │───►│   KYC    │───►│  Account ││
│  │          │    │  Contact │    │   KYC    │    │  Review  │    │  Active  ││
│  │          │    │          │    │          │    │          │    │          ││
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘│
│       │               │               │               │               │       │
│       ▼               ▼               ▼               ▼               ▼       │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐│
│  │ Email/   │    │ OTP      │    │ ID, KRA  │    │ Staff    │    │ Can now  ││
│  │ Phone    │    │ Verify   │    │ PIN,     │    │ verifies │    │ apply    ││
│  │ provided │    │ email &  │    │ Proof of │    │ documents│    │ for      ││
│  │          │    │ phone    │    │ Address  │    │          │    │ loans    ││
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘│
│                                                                                 │
│  Platform: Laravel Portal / Mobile App                                         │
│  Verification: Automated + Manual Review                                       │
│  Timeline: 24-48 hours                                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Loan Application Process

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          LOAN APPLICATION PROCESS                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  CLIENT ACTIONS                          STAFF ACTIONS                         │
│  ──────────────                          ─────────────                         │
│                                                                                 │
│  ┌──────────────────┐                                                          │
│  │ 1. Select        │                                                          │
│  │    Product       │                                                          │
│  └────────┬─────────┘                                                          │
│           │                                                                     │
│           ▼                                                                     │
│  ┌──────────────────┐                                                          │
│  │ 2. Add           │                                                          │
│  │    Beneficiary   │                                                          │
│  └────────┬─────────┘                                                          │
│           │                                                                     │
│           ▼                                                                     │
│  ┌──────────────────┐                                                          │
│  │ 3. Enter Loan    │                                                          │
│  │    Details       │                                                          │
│  └────────┬─────────┘                                                          │
│           │                                                                     │
│           ▼                                                                     │
│  ┌──────────────────┐                                                          │
│  │ 4. Register      │                                                          │
│  │    Collateral    │                                                          │
│  └────────┬─────────┘                                                          │
│           │                                                                     │
│           ▼                                                                     │
│  ┌──────────────────┐                    ┌──────────────────┐                  │
│  │ 5. Submit        │───────────────────►│ 6. Application   │                  │
│  │    Application   │                    │    Review        │                  │
│  └──────────────────┘                    └────────┬─────────┘                  │
│                                                   │                            │
│                                                   ▼                            │
│                                          ┌──────────────────┐                  │
│                                          │ 7. Credit        │                  │
│                                          │    Assessment    │                  │
│                                          │    (via CBS)     │                  │
│                                          └────────┬─────────┘                  │
│                                                   │                            │
│                                          ┌───────┴───────┐                     │
│                                          │               │                     │
│                                          ▼               ▼                     │
│                                   ┌────────────┐ ┌────────────┐                │
│                                   │  Approved  │ │  Rejected  │                │
│                                   └──────┬─────┘ └──────┬─────┘                │
│                                          │              │                      │
│                                          ▼              ▼                      │
│  ┌──────────────────┐            ┌────────────┐ ┌────────────┐                │
│  │ 8. Sign          │◄───────────│ Contract   │ │ Rejection  │                │
│  │    Contract      │            │ Generated  │ │ Notice     │                │
│  └────────┬─────────┘            └────────────┘ └────────────┘                │
│           │                                                                     │
│           ▼                                                                     │
│  ┌──────────────────┐                    ┌──────────────────┐                  │
│  │ 9. Receive       │◄───────────────────│ 10. Disbursement │                  │
│  │    Funds         │                    │     to School    │                  │
│  └──────────────────┘                    └──────────────────┘                  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.3 Payment & Reconciliation

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        PAYMENT & RECONCILIATION FLOW                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐│
│  │  Client  │    │  M-Pesa  │    │  EduFin  │    │   Core   │    │  Client  ││
│  │  Makes   │───►│  Process │───►│  Webhook │───►│  Banking │───►│  Account ││
│  │  Payment │    │          │    │  Handler │    │  Update  │    │  Updated ││
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────────┘│
│                                                                                 │
│  Daily Reconciliation:                                                         │
│  • Automated job compares EduFin records with CBS                             │
│  • Discrepancies flagged for manual review                                    │
│  • Reports generated for finance team                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Data Architecture

### 7.1 Data Ownership

| Data Category | Owner | Storage | Access |
|---------------|-------|---------|--------|
| Marketing Content | WordPress | MySQL | Public |
| User Accounts | Laravel | PostgreSQL | Authenticated |
| KYC Documents | Laravel | Cloudflare R2 | Authorized Staff + Owner |
| Loan Data | Laravel + CBS | PostgreSQL + CBS | Authorized Staff + Owner |
| Financial Transactions | CBS | CBS Database | Via CBS API |
| Audit Logs | Laravel | PostgreSQL | System Admin |

### 7.2 Data Flow Principles

1. **WordPress → Laravel (Read Only)**
   - Public data: Financing packages, interest rates
   - SSO authentication tokens
   - No write access to Laravel data

2. **Laravel → Core Banking (Bidirectional)**
   - Outbound: Loan creation, disbursement requests
   - Inbound: Payment notifications, status updates
   - Secured via mTLS and request signing

3. **Mobile App → Laravel API (Bidirectional)**
   - Same data access as web portal
   - JWT authentication
   - Rate limited

### 7.3 Core Entities

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              CORE DATA ENTITIES                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐  │
│  │  Account Holder │────────►│  Loan Facility  │◄────────│   Collateral    │  │
│  │                 │    1:N  │                 │    N:1  │                 │  │
│  │  • Personal Info│         │  • Amount       │         │  • Type         │  │
│  │  • KYC Status   │         │  • Term         │         │  • Value        │  │
│  │  • Contact      │         │  • Status       │         │  • Status       │  │
│  └────────┬────────┘         └────────┬────────┘         └─────────────────┘  │
│           │                           │                                        │
│           │ 1:N                       │ 1:N                                    │
│           ▼                           ▼                                        │
│  ┌─────────────────┐         ┌─────────────────┐                              │
│  │   Education     │         │    Obligor      │                              │
│  │   Beneficiary   │         │   Assignment    │                              │
│  │                 │         │                 │                              │
│  │  • Student Info │         │  • Role Type    │                              │
│  │  • School       │         │  • Liability %  │                              │
│  │  • Grade Level  │         │  • Status       │                              │
│  └─────────────────┘         └─────────────────┘                              │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Integration Landscape

### 8.1 Integration Map

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           INTEGRATION LANDSCAPE                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                              ┌─────────────────┐                               │
│                              │    LARAVEL      │                               │
│                              │    CORE APP     │                               │
│                              └────────┬────────┘                               │
│                                       │                                        │
│         ┌─────────────────────────────┼─────────────────────────────┐         │
│         │                             │                             │         │
│         ▼                             ▼                             ▼         │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐ │
│  │   WORDPRESS     │         │  CORE BANKING   │         │   MOBILE APP    │ │
│  │   (SSO, Data)   │         │    SYSTEM       │         │   (Flutter)     │ │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘ │
│                                       │                                        │
│         ┌─────────────────────────────┼─────────────────────────────┐         │
│         │                             │                             │         │
│         ▼                             ▼                             ▼         │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐ │
│  │    M-PESA       │         │  AFRICA'S       │         │   SENDGRID      │ │
│  │   (Payments)    │         │  TALKING (SMS)  │         │   (Email)       │ │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘ │
│                                                                                 │
│         ┌─────────────────────────────┬─────────────────────────────┐         │
│         │                             │                             │         │
│         ▼                             ▼                             ▼         │
│  ┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐ │
│  │     NTSA        │         │  LAND REGISTRY  │         │  CLOUDFLARE R2  │ │
│  │ (Vehicle Check) │         │ (Title Search)  │         │   (Storage)     │ │
│  └─────────────────┘         └─────────────────┘         └─────────────────┘ │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Integration Summary

| Integration | Direction | Protocol | Purpose |
|-------------|-----------|----------|---------|
| WordPress | Bidirectional | HTTPS + API Key | SSO, public data |
| Core Banking | Bidirectional | HTTPS + mTLS | Loan operations |
| M-Pesa | Inbound (webhook) | HTTPS | Payment processing |
| Africa's Talking | Outbound | HTTPS | SMS notifications |
| SendGrid | Outbound | HTTPS | Email notifications |
| NTSA | Outbound | HTTPS | Vehicle verification |
| Cloudflare R2 | Bidirectional | HTTPS | Document storage |
| Mobile App | Bidirectional | HTTPS + JWT | Client access |

---

## 9. Security & Compliance

### 9.1 Security Architecture

| Layer | Controls |
|-------|----------|
| **Edge** | Cloudflare WAF, DDoS protection, rate limiting |
| **Transport** | TLS 1.2+, mTLS for banking |
| **Application** | CSRF, XSS prevention, input validation |
| **Data** | Encryption at rest, PII encryption |
| **Access** | RBAC, MFA, session management |
| **Audit** | Comprehensive logging, 7-year retention |

### 9.2 Compliance Framework

| Requirement | Implementation |
|-------------|----------------|
| Data Protection Act | Consent management, data minimization, encryption |
| CBK Guidelines | Via Core Banking System partnership |
| AML/CFT | KYC verification, transaction monitoring |
| PCI-DSS | No card data stored; payments via M-Pesa |

> **See:** [Security Documentation](./security/README.md)

---

## 10. Project Roadmap

### 10.1 Phase Overview

| Phase | Timeline | Deliverables |
|-------|----------|--------------|
| **Phase 1: Foundation** | Months 1-3 | Core platform, basic loan processing |
| **Phase 2: Enhancement** | Months 4-6 | Advanced features, mobile app |
| **Phase 3: Scale** | Months 7-12 | Performance optimization, partnerships |

### 10.2 Milestone Summary

**Phase 1: Foundation**
- [ ] WordPress landing page live
- [ ] Laravel portal with client onboarding
- [ ] Basic loan application workflow
- [ ] Core banking integration
- [ ] Staff admin panel

**Phase 2: Enhancement**
- [ ] Mobile app (Flutter) launch
- [ ] Advanced collateral management
- [ ] Automated KYC verification
- [ ] Enhanced reporting

**Phase 3: Scale**
- [ ] Performance optimization
- [ ] Multi-institution support
- [ ] API partnerships
- [ ] Advanced analytics

---

## Document References

| Document | Purpose |
|----------|---------|
| [Implementation Guide](./implementation.md) | Technical implementation details |
| [Architecture Overview](./architecture/overview.md) | System architecture |
| [WordPress Architecture](./architecture/wordpress/README.md) | WordPress component |
| [Laravel Architecture](./architecture/laravel/README.md) | Laravel component |
| [API Documentation](./api/README.md) | API reference |
| [Security Documentation](./security/README.md) | Security protocols |

---

*This document provides the global overview of the EduFin platform. For technical implementation details, see the [Implementation Guide](./implementation.md).*
