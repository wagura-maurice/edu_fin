# Laravel Architecture

## Core Application Engine

**Version:** 2.0  
**Last Updated:** August 10, 2026

---

## Overview

Laravel serves as the **central engine** of the EduFin platform, powering all business operations, client interactions, and external integrations.

## Role Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    LARAVEL - CORE APPLICATION ENGINE                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     INTERNAL OPERATIONS                                  │  │
│  │                     (Staff via Filament Admin)                          │  │
│  │                                                                          │  │
│  │  • User & Role Management          • Loan Application Review            │  │
│  │  • KYC Verification Queue          • Collateral Approval                │  │
│  │  • Disbursement Processing         • Reports & Analytics                │  │
│  │  • System Configuration            • Audit Log Access                   │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     EXTERNAL CLIENT PORTAL                               │  │
│  │                     (Clients via Livewire)                              │  │
│  │                                                                          │  │
│  │  • Account Dashboard               • Loan Applications                  │  │
│  │  • Profile & KYC Management        • Beneficiary Management             │  │
│  │  • Document Upload                 • Payment Tracking                   │  │
│  │  • Statement Generation            • Notification Center                │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     API LAYER                                            │  │
│  │                     (Mobile & Integrations)                             │  │
│  │                                                                          │  │
│  │  • REST API for Flutter App        • Webhook Receivers                  │  │
│  │  • JWT Authentication              • API Versioning                     │  │
│  │  • Rate Limiting                                                        │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                     BANKING INTEGRATION LAYER                            │  │
│  │                     (Core Banking System)                               │  │
│  │                                                                          │  │
│  │  • Loan Account Creation           • Disbursement Requests              │  │
│  │  • Payment Processing              • Balance Inquiries                  │  │
│  │  • Statement Retrieval             • Reconciliation                     │  │
│  │  • Webhook Processing              • Error Handling                     │  │
│  │                                                                          │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Technology Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| Laravel | 11.x | Application framework |
| PHP | 8.3 | Runtime |
| Livewire | 3.x | Portal UI components |
| Filament | 3.x | Admin panel |
| PostgreSQL | 16 | Primary database |
| Redis | 7.x | Cache, sessions, queues |
| Horizon | Latest | Queue management |
| Sanctum | Latest | API authentication |

## Application Components

### 1. Client Portal (Livewire)

**URL:** `app.edufin.co.ke`

**Features:**
- Dashboard with account overview
- Profile management & KYC submission
- Loan application workflow
- Beneficiary management
- Document upload & management
- Payment schedule & history
- Statement generation
- Notification center

### 2. Admin Panel (Filament)

**URL:** `app.edufin.co.ke/admin`

**Features:**
- User & role management
- Loan application review queue
- KYC verification workflow
- Collateral approval
- Disbursement processing
- Reports & analytics
- System configuration
- Audit log viewer

### 3. REST API

**URL:** `edufin.co.ke/api/v1`

**Consumers:**
- WordPress onboarding wizard (registration + dynamic options) — **primary consumer**
- Flutter mobile app (iOS/Android)
- Future partner integrations

**Authentication:**
- JWT (Sanctum) for mobile
- Signature verification for webhooks

### 4. Background Services (Horizon)

**Jobs:**
- `ProcessLoanApplication` - Async loan processing
- `SyncLoanWithCoreBanking` - CBS synchronization
- `ProcessPaymentWebhook` - Payment processing
- `SendNotification` - Email/WhatsApp dispatch
- `GenerateStatement` - PDF generation
- `ReconcilePayments` - Daily reconciliation

## Service Layer

```
app/Services/
├── Banking/
│   ├── CoreBankingService.php      # CBS communication
│   ├── PaymentService.php          # Payment processing
│   └── ReconciliationService.php   # Daily reconciliation
│
├── Kyc/
│   └── KycVerificationService.php  # KYC workflow
│
├── Loan/
│   ├── LoanApplicationService.php  # Application processing
│   ├── LoanCalculatorService.php   # Calculations
│   └── StatementService.php        # Statement generation
│
├── Notification/
│   ├── EmailService.php            # Email dispatch
│   ├── WhatsappService.php         # WhatsApp dispatch (via WAHA)
│   └── PushNotificationService.php # Push notifications
│
└── Document/
    └── DocumentStorageService.php  # R2 storage
```

## Database Schema

**Primary Tables:**
- `users` - Authentication
- `account_holders` - Client profiles
- `education_beneficiaries` - Students
- `loan_facilities` - Loan accounts
- `obligor_assignments` - Liability tracking
- `collaterals` - Collateral records
- `documents` - Document metadata
- `payments` - Payment records
- `audit_logs` - Audit trail

## Registration Architecture (API-Only)

The Laravel application does **not** host a front-end `/register` page. Registration is handled exclusively via the REST API, following an API-first design that separates the onboarding UI from the data logic.

### Responsibilities

| Layer | Owner | Responsibility |
|-------|-------|----------------|
| Onboarding UI | WordPress (`edufin.co.ke/get-started`) | Multi-step wizard, client-side validation, user experience |
| Registration data logic | Laravel API (`edufin.co.ke/api/v1/auth/register`) | Server-side validation, user creation, business rules |
| Dynamic options | Laravel API (`edufin.co.ke/api/v1/options/*`) | Locations, genders, employment types, income ranges, education levels, relationship types |

### Laravel UI (Web Routes)

The Laravel web UI is intentionally limited to authentication and password management only:

| Route | Purpose |
|-------|---------|
| `/login` | Unified login (clients + staff), role-based redirect |
| `/forgot-password` | Password reset request (email link) |
| `/password/reset` | Password reset form (link landing) |
| `/logout` | End session |

> **No `/register` route.** This prevents a duplicate onboarding surface and keeps all registration logic centralized in the API layer.

### Registration API Endpoint

```
POST https://edufin.co.ke/api/v1/auth/register
```

**Consumers:**
- WordPress onboarding wizard (primary, today)
- Flutter mobile app (future)

**Request:** Client registration data (personal details, employment, beneficiary, credentials).
**Response:** JWT token + account ID on success; validation errors on failure.

### Dynamic Options Endpoints

The wizard's dropdown fields are populated from Laravel so they always reflect the latest backend data without WordPress code changes:

| Endpoint | Options |
|----------|---------|
| `GET /api/v1/options/locations` | Counties/cities |
| `GET /api/v1/options/genders` | Gender options |
| `GET /api/v1/options/employment-types` | Employment types |
| `GET /api/v1/options/income-ranges` | Income ranges |
| `GET /api/v1/options/education-levels` | Education levels |
| `GET /api/v1/options/relationship-types` | Beneficiary relationship types |

## Security Implementation

| Layer | Implementation |
|-------|----------------|
| Authentication | Session (web), JWT (API) |
| Authorization | Policies, Gates, RBAC |
| Input Validation | Form Requests |
| Output Encoding | Blade auto-escape |
| CSRF | Token verification |
| Rate Limiting | Throttle middleware |
| Encryption | PII encrypted at rest |
| Audit | All changes logged |

## External Integrations

| Service | Purpose | Protocol |
|---------|---------|----------|
| Core Banking | Financial operations | HTTPS + mTLS |
| M-Pesa | Payments | HTTPS + OAuth |
| WAHA | WhatsApp | HTTPS |
| SendGrid | Email | HTTPS |
| Cloudflare R2 | Document storage | HTTPS |
| NTSA | Vehicle verification | HTTPS |

---

**See Also:**
- [Portal Documentation](./portal.md)
- [API Documentation](./api.md)
- [Services Documentation](./services.md)
- [Banking Integration](./banking-integration.md)
