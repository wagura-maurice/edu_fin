# EduFin - Implementation Documentation

## Technical Implementation Guide

**Version:** 1.0  
**Last Updated:** August 6, 2026  
**Classification:** Technical Documentation

---

## Table of Contents

1. [Implementation Overview](#1-implementation-overview)
2. [WordPress Implementation](#2-wordpress-implementation)
3. [Laravel Implementation](#3-laravel-implementation)
4. [Database Architecture](#4-database-architecture)
5. [API Implementation](#5-api-implementation)
6. [Authentication & SSO](#6-authentication--sso)
7. [Core Banking Integration](#7-core-banking-integration)
8. [Security Implementation](#8-security-implementation)
9. [Deployment & Infrastructure](#9-deployment--infrastructure)
10. [Testing Strategy](#10-testing-strategy)

---

## 1. Implementation Overview

### 1.1 Architecture Summary

The EduFin platform is implemented as two distinct, loosely-coupled systems:

| Component | Purpose | Technology | Database |
|-----------|---------|------------|----------|
| **WordPress** | Standalone landing page | WordPress 6.x, PHP 8.2 | MySQL (content only) |
| **Laravel** | Core application engine | Laravel 11.x, PHP 8.3 | PostgreSQL (all business data) |

### 1.2 Repository Structure

```
edufin/
├── docs/                           # Documentation
│   ├── project_documentation.md    # Global overview
│   ├── implementation.md           # This file
│   └── architecture/               # Architecture docs
│
├── wordpress/                      # WordPress Installation
│   ├── wp-content/
│   │   ├── themes/edufin/         # Custom theme
│   │   └── plugins/
│   │       ├── edufin-sso/        # SSO integration
│   │       └── edufin-api/        # API client
│   └── wp-config.php
│
├── laravel/                        # Laravel Application
│   ├── app/
│   │   ├── Http/Controllers/Api/  # REST API
│   │   ├── Http/Livewire/         # Portal UI
│   │   ├── Filament/              # Admin panel
│   │   ├── Services/              # Business logic
│   │   └── Models/                # Eloquent models
│   ├── routes/
│   │   ├── web.php                # Portal routes
│   │   └── api.php                # API routes
│   └── config/
│
├── infrastructure/                 # Infrastructure as Code
│   ├── docker/
│   └── nginx/
│
└── shared/                         # Shared Assets (CDN)
    ├── css/
    └── images/
```

### 1.3 Communication Patterns

```
WordPress ──► Laravel API (HTTPS + API Key) ──► Read public data, SSO
Laravel ──► Core Banking (HTTPS + mTLS) ──► Financial operations
Mobile App ──► Laravel API (HTTPS + JWT) ──► Full client functionality
```

---

## 2. WordPress Implementation

### 2.1 Purpose & Scope

WordPress serves **exclusively** as the company landing page:
- Decoupled from core application logic
- Managed by secretarial staff
- Contains NO business data or PII
- Communicates with Laravel only for SSO and public data

### 2.2 Theme Structure

```
wordpress/wp-content/themes/edufin/
├── style.css                    # Theme metadata & styles
├── functions.php                # Theme setup & hooks
├── header.php                   # Site header
├── footer.php                   # Site footer
├── front-page.php               # Homepage template
├── page.php                     # Generic page template
├── single.php                   # Blog post template
├── archive.php                  # Blog archive
├── template-parts/
│   ├── header/nav.php
│   ├── footer/widgets.php
│   └── content/
├── inc/
│   ├── customizer.php           # Theme customizer
│   └── template-functions.php
└── assets/
    ├── css/
    ├── js/
    └── images/
```

### 2.3 Custom Plugins

#### EduFin SSO Plugin
Handles Single Sign-On with Laravel portal.

**Key Functions:**
- `check_sso_session()` - Validates SSO cookie on each request
- `handle_sso_callback()` - Processes redirect from Laravel with token
- `custom_login_url()` - Redirects login to Laravel SSO

#### EduFin API Plugin
Fetches public data from Laravel API.

**Key Functions:**
- `get_packages()` - Fetch financing packages
- `get_calculator_rates()` - Fetch loan calculator rates
- `submit_inquiry()` - Submit contact form to Laravel

**Shortcodes:**
- `[edufin_products]` - Display financing packages
- `[edufin_calculator]` - Loan calculator widget
- `[edufin_login_button]` - Login/portal button

### 2.4 Configuration

```php
// wp-config.php

// EduFin Integration
define('EDUFIN_API_URL', 'https://api.edufin.co.ke');
define('EDUFIN_API_KEY', getenv('EDUFIN_API_KEY'));
define('EDUFIN_PORTAL_URL', 'https://app.edufin.co.ke');

// Security
define('DISALLOW_FILE_EDIT', true);
define('DISALLOW_FILE_MODS', true);
define('FORCE_SSL_ADMIN', true);

// Database prefix (non-default)
$table_prefix = 'edf_';
```

> **Full WordPress documentation:** [architecture/wordpress/README.md](./architecture/wordpress/README.md)

---

## 3. Laravel Implementation

### 3.1 Purpose & Scope

Laravel serves as the **core application engine**:

1. **Internal Operations** (Staff via Filament Admin)
   - User management
   - Loan application review
   - KYC verification
   - Reporting & analytics

2. **External Client Portal** (Clients via Livewire)
   - Self-service dashboard
   - Loan applications
   - Document management
   - Payment tracking

3. **API Layer** (Mobile & Integrations)
   - REST API for Flutter app
   - WordPress integration endpoints
   - Webhook receivers

4. **Banking Integration**
   - Core Banking System communication
   - Payment processing
   - Reconciliation

### 3.2 Application Structure

```
laravel/app/
├── Console/Commands/           # Artisan commands
│   ├── ReconcilePayments.php
│   └── SyncWithCoreBanking.php
│
├── Events/                     # Domain events
│   ├── LoanApplied.php
│   ├── LoanApproved.php
│   └── PaymentReceived.php
│
├── Filament/                   # Admin panel (Filament 3.x)
│   ├── Resources/
│   │   ├── UserResource.php
│   │   ├── LoanApplicationResource.php
│   │   └── CollateralResource.php
│   └── Widgets/
│
├── Http/
│   ├── Controllers/
│   │   ├── Api/V1/            # REST API controllers
│   │   │   ├── AuthController.php
│   │   │   ├── LoanController.php
│   │   │   └── PublicController.php
│   │   └── Sso/
│   │       └── SsoController.php
│   │
│   ├── Livewire/              # Portal components
│   │   └── Portal/
│   │       ├── Dashboard.php
│   │       ├── Loans/
│   │       └── Profile/
│   │
│   ├── Middleware/
│   │   ├── VerifyApiKey.php
│   │   ├── EnsureKycComplete.php
│   │   └── LogApiAccess.php
│   │
│   └── Requests/              # Form validation
│
├── Jobs/                       # Queue jobs
│   ├── ProcessLoanApplication.php
│   ├── SyncLoanWithCoreBanking.php
│   └── SendNotification.php
│
├── Models/                     # Eloquent models
│   ├── User.php
│   ├── AccountHolder.php
│   ├── EducationBeneficiary.php
│   ├── LoanFacility.php
│   ├── ObligorAssignment.php
│   ├── Collateral.php
│   └── Document.php
│
├── Policies/                   # Authorization
│   ├── LoanPolicy.php
│   └── DocumentPolicy.php
│
└── Services/                   # Business logic
    ├── Banking/
    │   ├── CoreBankingService.php
    │   └── ReconciliationService.php
    ├── Kyc/
    │   └── KycVerificationService.php
    ├── Loan/
    │   └── LoanApplicationService.php
    ├── Notification/
    │   ├── EmailService.php
    │   └── SmsService.php
    └── Sso/
        └── SsoService.php
```

### 3.3 Key Services

#### SsoService
Manages SSO tokens and sessions for WordPress integration.

#### CoreBankingService
Handles all communication with Core Banking System:
- Loan account creation
- Disbursement requests
- Balance inquiries
- Statement retrieval

#### LoanApplicationService
Processes loan applications:
- Validation
- Credit scoring (via CBS)
- Approval workflow
- Contract generation

> **Full Laravel documentation:** [architecture/laravel/README.md](./architecture/laravel/README.md)

---

## 4. Database Architecture

### 4.1 Database Separation

| Database | Platform | Purpose |
|----------|----------|---------|
| MySQL | WordPress | Content only (posts, pages, media) |
| PostgreSQL | Laravel | All business data (users, loans, KYC) |

**Rule:** WordPress database contains NO PII or financial data.

### 4.2 Core Tables (PostgreSQL)

| Table | Purpose |
|-------|---------|
| `users` | Authentication credentials |
| `account_holders` | Client profiles, KYC data |
| `education_beneficiaries` | Students/dependents |
| `loan_facilities` | Loan accounts |
| `obligor_assignments` | Liability assignments |
| `collaterals` | Collateral records |
| `documents` | Document metadata |
| `payments` | Payment records |
| `audit_logs` | Comprehensive audit trail |
| `sso_tokens` | One-time SSO tokens |
| `sso_sessions` | Persistent SSO sessions |

### 4.3 Entity Relationships

```
AccountHolder (1) ──► (N) EducationBeneficiary
AccountHolder (1) ──► (N) LoanFacility (via ObligorAssignment)
LoanFacility (1) ──► (N) ObligorAssignment
LoanFacility (1) ──► (1) EducationBeneficiary
LoanFacility (N) ◄── (1) Collateral
AccountHolder (1) ──► (N) Document
LoanFacility (1) ──► (N) Document
```

---

## 5. API Implementation

### 5.1 API Structure

```
api.edufin.co.ke/api/v1/
│
├── /public (API key auth)
│   ├── GET  /packages
│   ├── GET  /calculator/rates
│   └── POST /inquiries
│
├── /sso (API key auth - WordPress)
│   ├── POST /validate
│   ├── POST /session
│   └── GET  /me
│
├── /auth
│   ├── POST /register
│   ├── POST /login
│   ├── POST /logout
│   └── POST /verify-otp
│
├── /profile (JWT auth)
│   ├── GET  /
│   ├── PUT  /
│   └── POST /kyc
│
├── /beneficiaries (JWT auth)
│   ├── GET  /
│   ├── POST /
│   ├── GET  /{id}
│   └── PUT  /{id}
│
├── /loans (JWT auth)
│   ├── GET  /
│   ├── POST /
│   ├── GET  /{id}
│   ├── GET  /{id}/schedule
│   └── GET  /{id}/statement
│
├── /collateral (JWT auth)
│   ├── GET  /
│   ├── POST /
│   └── GET  /{id}
│
├── /documents (JWT auth)
│   ├── GET  /
│   ├── POST /upload
│   └── GET  /{id}
│
└── /webhooks (Signature verification)
    ├── POST /banking/payment
    ├── POST /banking/status
    └── POST /mpesa/callback
```

### 5.2 Authentication Methods

| Endpoint Type | Method | Header |
|---------------|--------|--------|
| Public | API Key | `X-API-Key: {key}` |
| SSO | API Key | `X-API-Key: {key}` |
| Authenticated | JWT | `Authorization: Bearer {token}` |
| Webhooks | Signature | `X-Webhook-Signature: {sig}` |

> **Full API documentation:** [api/README.md](./api/README.md)

---

## 6. Authentication & SSO

### 6.1 Authentication Architecture

Laravel serves as the **Identity Provider** for all platforms:

| Platform | Auth Method | Storage |
|----------|-------------|---------|
| Portal (Web) | Session | Redis |
| Mobile App | JWT (Sanctum) | Stateless |
| WordPress | SSO Token | Redis |

### 6.2 SSO Flow

1. User clicks "Login" on WordPress
2. WordPress redirects to `app.edufin.co.ke/sso/login`
3. User authenticates on Laravel
4. Laravel generates one-time token + SSO session
5. Laravel redirects back to WordPress with token
6. WordPress validates token via API
7. WordPress creates local session

### 6.3 Key Components

- `SsoService` - Token/session management
- `SsoController` - SSO endpoints
- WordPress `EduFin_SSO` plugin - WordPress integration

> **Full SSO implementation:** [IMPLEMENTATION_PLAN.md](../IMPLEMENTATION_PLAN.md) Section 3

---

## 7. Core Banking Integration

### 7.1 Integration Pattern

Asynchronous with synchronous fallback:

1. Laravel initiates request to CBS
2. CBS processes and responds (or queues)
3. CBS sends webhook for async operations
4. Laravel processes webhook and updates records

### 7.2 Security Layers

| Layer | Implementation |
|-------|----------------|
| Network | VPN tunnel / IP whitelist |
| Transport | mTLS (mutual TLS) |
| Application | API key + HMAC-SHA256 signing |
| Data | Request/response encryption |

### 7.3 Operations

**Outbound (EduFin → CBS):**
- Create loan account
- Request disbursement
- Get balance
- Get statement

**Inbound (CBS → EduFin):**
- Payment received
- Loan status update
- Disbursement confirmation

> **Full banking integration:** [architecture/laravel/banking-integration.md](./architecture/laravel/banking-integration.md)

---

## 8. Security Implementation

### 8.1 Security Layers

| Layer | WordPress | Laravel |
|-------|-----------|---------|
| Edge | Cloudflare WAF | Cloudflare WAF |
| Transport | TLS 1.2+ | TLS 1.2+ / mTLS |
| Application | Limited plugins | CSRF, XSS, SQLi prevention |
| Data | No PII | Encryption at rest |
| Access | Role-based | RBAC + Policies |
| Audit | Basic logging | Comprehensive audit |

### 8.2 Data Protection

- PII encrypted at application level
- Passwords hashed with Argon2id
- Sessions stored in Redis (encrypted)
- Documents stored in R2 with signed URLs

> **Full security documentation:** [security/README.md](./security/README.md)

---

## 9. Deployment & Infrastructure

### 9.1 Infrastructure Overview

| Component | Specification |
|-----------|---------------|
| WordPress Server | 2 vCPU, 4GB RAM |
| Laravel Server | 4 vCPU, 8GB RAM |
| PostgreSQL | Managed, 2 vCPU, 4GB |
| MySQL | Managed, 1 vCPU, 2GB |
| Redis | Managed, 256MB |

### 9.2 Deployment Process

**WordPress:**
1. Git pull changes
2. Clear cache (WP Rocket)
3. Verify plugins

**Laravel:**
1. Git pull changes
2. `composer install --no-dev`
3. `php artisan migrate --force`
4. `php artisan config:cache`
5. `php artisan route:cache`
6. `php artisan queue:restart`

> **Full deployment guide:** [deployment/README.md](./deployment/README.md)

---

## 10. Testing Strategy

### 10.1 Test Types

| Type | WordPress | Laravel |
|------|-----------|---------|
| Unit | PHPUnit (plugins) | PHPUnit |
| Feature | - | PHPUnit |
| Integration | - | PHPUnit |
| E2E | Cypress | Cypress |
| API | - | Postman/PHPUnit |

### 10.2 Test Coverage Targets

| Component | Target |
|-----------|--------|
| Services | 90% |
| Controllers | 80% |
| Models | 70% |
| Livewire | 70% |

---

## Document References

| Document | Purpose |
|----------|---------|
| [Project Documentation](./project_documentation.md) | Global overview |
| [WordPress Architecture](./architecture/wordpress/README.md) | WordPress details |
| [Laravel Architecture](./architecture/laravel/README.md) | Laravel details |
| [API Documentation](./api/README.md) | API reference |
| [Security Documentation](./security/README.md) | Security protocols |
| [Deployment Guide](./deployment/README.md) | Infrastructure |

---

*This document provides the technical implementation guide for the EduFin platform. For business context and project overview, see the [Project Documentation](./project_documentation.md).*
