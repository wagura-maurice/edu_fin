# EduFin Platform - Technical Architecture Specification

## Dual-Platform Ecosystem: WordPress + Laravel/Livewire

**Date:** August 6, 2026  
**Version:** 1.0  
**Classification:** Technical Architecture Document

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [System Architecture Overview](#2-system-architecture-overview)
3. [Platform Responsibilities & Delineation](#3-platform-responsibilities--delineation)
4. [Data Flow Architecture](#4-data-flow-architecture)
5. [Security Architecture](#5-security-architecture)
6. [Role-Based Access Control (RBAC)](#6-role-based-access-control-rbac)
7. [API Architecture & Banking Integration](#7-api-architecture--banking-integration)
8. [Separation of Concerns Analysis](#8-separation-of-concerns-analysis)
9. [Mobile Ecosystem Transition Strategy](#9-mobile-ecosystem-transition-strategy)
10. [Infrastructure Architecture](#10-infrastructure-architecture)
11. [Integration Points & Contracts](#11-integration-points--contracts)
12. [Scalability & Performance](#12-scalability--performance)

---

## 1. Executive Summary

This document defines the technical architecture for the EduFin dual-platform ecosystem, comprising:

1. **WordPress Layer** - Marketing, content management, and public-facing brand presence
2. **Laravel/Livewire Layer** - Secure application portal for client onboarding and transaction management

### Architecture Principles

| Principle | Implementation |
|-----------|----------------|
| **Separation of Concerns** | Content management isolated from transaction processing |
| **Security by Design** | Financial operations in hardened Laravel environment |
| **API-First** | All business logic exposed via versioned REST APIs |
| **Scalability** | Stateless services, horizontal scaling capability |
| **Mobile-Ready** | Architecture designed for Flutter app integration |

### Platform Summary

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              EDUFIN PLATFORM ECOSYSTEM                              │
├─────────────────────────────────┬───────────────────────────────────────────────────┤
│   WORDPRESS                     │   LARAVEL/LIVEWIRE                               │
│   Content & Marketing Layer     │   Application & Transaction Layer                │
│                                 │                                                   │
│   • Company Profile             │   • Client Registration & Onboarding             │
│   • Product Catalog             │   • Dependent Management                         │
│   • Blog & Articles             │   • Loan Application Processing                  │
│   • FAQs & Help Center          │   • KYC Document Management                      │
│   • Testimonials                │   • Collateral Registration                      │
│   • Contact Information         │   • Statement Generation                         │
│   • Legal Documents             │   • Payment Tracking                             │
│   • Newsletter Management       │   • Core Banking Integration                     │
│   • Live Chat Widget            │   • Notification Services                        │
│   • SEO Optimization            │   • Audit & Compliance Logging                   │
│                                 │                                                   │
│   Audience: Public              │   Audience: Authenticated Users                  │
│   Admin: Secretarial Staff      │   Admin: System Administrators                   │
│                                 │   Users: Clients, Dependents, Staff              │
└─────────────────────────────────┴───────────────────────────────────────────────────┘
```

---

## 2. System Architecture Overview

### 2.1 High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                        INTERNET / USERS                                              │
│                                                                                                     │
│    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                   │
│    │   Browser    │    │   Browser    │    │   Flutter    │    │   Flutter    │                   │
│    │  (Public)    │    │  (Portal)    │    │  (iOS App)   │    │(Android App) │                   │
│    └──────┬───────┘    └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                   │
└───────────┼───────────────────┼───────────────────┼───────────────────┼────────────────────────────┘
            │                   │                   │                   │
            ▼                   ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                         CLOUDFLARE                                                   │
│   • DNS Management          • WAF (Web Application Firewall)      • Bot Protection                 │
│   • SSL/TLS Termination     • DDoS Mitigation                     • Rate Limiting                  │
│   • CDN (Static Assets)     • Geo-blocking (if required)          • Access Rules                   │
│                                                                                                     │
│         │ edufin.co.ke              │ app.edufin.co.ke                │ api.edufin.co.ke          │
└─────────┬───────────────────────────┬─────────────────────────────────┬───────────────────────────┘
          │                           │                                 │
          ▼                           └─────────────────┬───────────────┘
┌─────────────────────────────────┐                     │
│   WORDPRESS SERVER              │                     ▼
│   (Content Management Layer)    │   ┌─────────────────────────────────────────────────────────────┐
│                                 │   │   LARAVEL APPLICATION CLUSTER                              │
│   • Nginx + PHP-FPM 8.2         │   │   (Application & Transaction Layer)                        │
│   • WordPress 6.x               │   │                                                             │
│   • Custom Theme (EduFin)       │   │   • Nginx (Load Balancer)                                  │
│   • Plugins:                    │   │   • PHP-FPM 8.3 Workers                                    │
│     - EduFin SSO                │   │   • Livewire (Portal UI)                                   │
│     - EduFin API Client         │   │   • REST API (Mobile/Integrations)                        │
│     - Yoast SEO                 │   │   • Filament (Admin Panel)                                │
│     - WP Rocket                 │   │   • Horizon (Queue Processing)                            │
│     - LiveChat                  │   │                                                             │
│     - Mailchimp                 │   │   Business Services:                                       │
│                                 │   │   • KYC Service                                            │
│   Database: MySQL 8.0           │   │   • Loan Service                                           │
│   (Content Only - No PII)       │   │   • Collateral Service                                     │
│                                 │   │   • Payment Service                                        │
└─────────────────────────────────┘   │   • Notification Service                                   │
                                      │   • Audit Service                                           │
          │                           │                                                             │
          │   Shared Redis            │   Database: PostgreSQL 16                                  │
          │   (SSO Sessions)          │   Cache: Redis                                             │
          └───────────────────────────┤                                                             │
                                      └─────────────────────────────────────────────────────────────┘
                                                        │
                                                        ▼
                                      ┌─────────────────────────────────────────────────────────────┐
                                      │                  EXTERNAL INTEGRATIONS                      │
                                      │                                                             │
                                      │   • Core Banking API        • Payment Gateways (M-Pesa)    │
                                      │   • Document Verification   • SMS (Africa's Talking)       │
                                      │   • Email (SendGrid)        • Storage (Cloudflare R2)      │
                                      └─────────────────────────────────────────────────────────────┘
```

### 2.2 Domain Architecture

| Domain | Platform | Purpose | Access |
|--------|----------|---------|--------|
| `edufin.co.ke` | WordPress | Public marketing website | Public |
| `www.edufin.co.ke` | WordPress | Redirect to root domain | Public |
| `app.edufin.co.ke` | Laravel/Livewire | Client portal (web) | Authenticated |
| `api.edufin.co.ke` | Laravel | REST API (mobile, integrations) | Authenticated |
| `admin.edufin.co.ke` | Laravel/Filament | Internal administration | Staff Only |
| `cdn.edufin.co.ke` | Cloudflare R2 | Static assets, media | Public |

---

## 3. Platform Responsibilities & Delineation

### 3.1 WordPress - Content & Marketing Layer

#### Purpose Statement
WordPress serves as the **public-facing brand presence** and **content management system**, optimized for SEO, marketing campaigns, and non-technical content administration by secretarial staff.

#### Functional Responsibilities

**Content Management:**
- Company Profile (About Us, Mission & Vision, Team, Locations)
- Product/Service Catalog (Financing packages, rates, eligibility)
- Blog & Articles (Financial education, news, success stories)
- FAQs (Categorized help content)
- Testimonials & Reviews
- Contact Information

**Marketing & SEO:**
- SEO Optimization (Meta tags, schema markup, sitemaps)
- Analytics & Tracking (GA4, GTM, conversion tracking)
- Email Marketing (Newsletter signup, Mailchimp integration)

**Customer Support:**
- Live Chat Integration (Tawk.to / LiveChat)
- Help Resources (Guides, video tutorials, downloadable forms)

**Legal & Compliance:**
- Terms & Conditions
- Privacy Policy
- Cookie Policy
- Data Protection Notice

#### Administrative Interface (Secretarial Staff)

```
WORDPRESS ADMIN - SECRETARIAL STAFF CAPABILITIES
─────────────────────────────────────────────────

ALLOWED:
├── Pages (Edit content, not structure)
├── Blog Posts (Create, edit, publish)
├── FAQs (Manage Q&A content)
├── Testimonials (Add, approve, manage)
├── Newsletter (View subscribers, send campaigns)
├── Contact Submissions (View, export, forward)
├── Media Library (Upload images, documents)
└── Menus (Limited editing)

RESTRICTED:
├── Plugins (View only, cannot install/modify)
├── Users (Cannot add/edit/delete)
├── Settings (View only)
├── Theme Editor (No access)
└── Tools (No access)
```

#### WordPress Plugin Architecture

| Plugin | Purpose | Configuration |
|--------|---------|---------------|
| **EduFin SSO** | Custom SSO integration with Laravel | API key, endpoints |
| **EduFin API Client** | Fetch data from Laravel (packages, rates) | API key, caching |
| **Yoast SEO** | SEO optimization | Auto-generate meta |
| **WP Rocket** | Performance caching | Page cache, CDN |
| **Tawk.to / LiveChat** | Live chat support | Business hours, routing |
| **Mailchimp for WP** | Newsletter integration | List sync, forms |
| **WPForms** | Contact forms | Spam protection, notifications |
| **Wordfence** | Security hardening | Firewall, malware scan |
| **UpdraftPlus** | Automated backups | Daily to Backblaze B2 |

### 3.2 Laravel/Livewire - Application & Transaction Layer

#### Purpose Statement
Laravel/Livewire serves as the **secure application portal** handling all sensitive operations including client onboarding, financial transactions, KYC processing, and core banking integration.

#### Functional Responsibilities

**Client Onboarding:**
- Registration Flow (Email/Phone verification, personal details)
- KYC Document Upload & Verification
- Identity Verification (National ID, KRA PIN)
- Account Verification Workflow

**Dependent Management:**
- Education Beneficiary Registration
- School Enrollment & Fee Structure
- Age Tracking & Liability Transitions
- Portal Access Management

**Loan Management:**
- Loan Application & Product Selection
- Credit Scoring (via Core Banking)
- Approval Workflow
- Contract Generation
- Disbursement Processing

**Collateral Management:**
- Asset Registration (Vehicle, Land, Property, Shares)
- Verification (NTSA, Land Registry)
- Valuation Reports
- Lien Registration

**Financial Operations:**
- Payment Tracking & History
- Statement Generation
- Balance Inquiries
- Overdue Management
- Core Banking Synchronization

**Communication:**
- Email Notifications
- SMS Alerts
- In-App Notifications
- Push Notifications (Mobile)

**Administration (Filament):**
- User Management
- Role Assignment
- Application Review
- System Configuration
- Reports & Analytics

**Compliance & Audit:**
- Comprehensive Audit Logging
- Data Change Tracking
- Access Logging
- Regulatory Reporting

#### Laravel Application Structure

```
laravel/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/V1/              # REST API controllers (versioned)
│   │   │   ├── Portal/              # Livewire web controllers
│   │   │   ├── Sso/                 # SSO controllers
│   │   │   └── Webhooks/            # External webhook handlers
│   │   ├── Livewire/                # Livewire components
│   │   │   ├── Dashboard/
│   │   │   ├── Onboarding/
│   │   │   ├── Loans/
│   │   │   ├── Beneficiaries/
│   │   │   └── Documents/
│   │   └── Middleware/
│   ├── Models/                      # Eloquent models
│   ├── Services/                    # Business logic services
│   │   ├── Banking/
│   │   ├── Kyc/
│   │   ├── Notification/
│   │   └── Document/
│   ├── Jobs/                        # Queue jobs
│   └── Events/                      # Event classes
├── routes/
│   ├── web.php                      # Livewire portal routes
│   ├── api.php                      # REST API routes
│   └── channels.php                 # WebSocket channels
└── config/
    ├── banking.php                  # Core banking config
    └── sso.php                      # SSO configuration
```

---

## 4. Data Flow Architecture

### 4.1 Data Flow Patterns

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    DATA FLOW PATTERNS                                                │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  FLOW 1: CONTENT BROWSING (WordPress → Laravel API → WordPress)                                    │
│  ─────────────────────────────────────────────────────────────────                                 │
│                                                                                                     │
│  User visits Products page → WordPress calls Laravel API → JSON response → WordPress renders       │
│                                                                                                     │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                      │
│  │  User    │────►│  WordPress   │────►│  Laravel     │────►│  WordPress   │                      │
│  │ (Browser)│     │  /products   │     │  /api/v1/    │     │  Renders     │                      │
│  └──────────┘     │              │     │  packages    │     │  Content     │                      │
│                   └──────────────┘     └──────────────┘     └──────────────┘                      │
│                                                                                                     │
│  FLOW 2: SSO AUTHENTICATION (WordPress → Laravel → WordPress)                                      │
│  ────────────────────────────────────────────────────────────                                      │
│                                                                                                     │
│  User clicks Login → Redirect to Laravel → Auth → Token → Redirect back → WP session              │
│                                                                                                     │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                      │
│  │  User    │────►│  WordPress   │────►│  Laravel     │────►│  WordPress   │                      │
│  │ (Login)  │     │  Redirect    │     │  /sso/login  │     │  Validate    │                      │
│  └──────────┘     └──────────────┘     │  Auth+Token  │     │  Create Sess │                      │
│                                        └──────────────┘     └──────────────┘                      │
│                                                                                                     │
│  FLOW 3: LOAN APPLICATION (Portal → Services → Database → Banking)                                 │
│  ─────────────────────────────────────────────────────────────────                                 │
│                                                                                                     │
│  Client submits → Livewire validates → Service processes → DB persist → Queue job → Banking API   │
│                                                                                                     │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐│
│  │  Client  │────►│  Livewire    │────►│  Loan        │────►│  PostgreSQL  │────►│  Core        ││
│  │ (Portal) │     │  Component   │     │  Service     │     │  + Queue Job │     │  Banking     ││
│  └──────────┘     └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘│
│                                                                                                     │
│  FLOW 4: MOBILE APP (Flutter → REST API → Services → Database)                                     │
│  ─────────────────────────────────────────────────────────────                                     │
│                                                                                                     │
│  App request → JWT auth → API controller → Service → Database → JSON response                      │
│                                                                                                     │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                      │
│  │  Flutter │────►│  Laravel     │────►│  Service     │────►│  PostgreSQL  │                      │
│  │  App     │     │  /api/v1/*   │     │  Layer       │     │              │                      │
│  │  (JWT)   │     │              │     │              │     │              │                      │
│  └──────────┘     └──────────────┘     └──────────────┘     └──────────────┘                      │
│                                                                                                     │
│  FLOW 5: WEBHOOK PROCESSING (External → Laravel → Database → Notifications)                        │
│  ──────────────────────────────────────────────────────────────────────────                        │
│                                                                                                     │
│  Banking webhook → Signature verify → Process → Update DB → Notify client                          │
│                                                                                                     │
│  ┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                      │
│  │  Core    │────►│  Laravel     │────►│  PostgreSQL  │────►│  Client      │                      │
│  │  Banking │     │  /webhook/   │     │  Update      │     │  Email/SMS   │                      │
│  │  (Event) │     │  payment     │     │  Records     │     │              │                      │
│  └──────────┘     └──────────────┘     └──────────────┘     └──────────────┘                      │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Data Flow Summary Table

| Flow | Source | Destination | Protocol | Data Type | Security |
|------|--------|-------------|----------|-----------|----------|
| Content Fetch | WordPress | Laravel API | HTTPS | JSON (packages, rates) | API Key |
| SSO Auth | WordPress | Laravel | HTTPS | Token exchange | One-time token |
| Session Validation | WordPress | Laravel API | HTTPS | Session ID | API Key + HMAC |
| Loan Application | Portal (Livewire) | PostgreSQL | Internal | Form data | CSRF + Auth |
| Banking Sync | Laravel | Core Banking | HTTPS + mTLS | JSON | Client cert + API key |
| Payment Webhook | Core Banking | Laravel | HTTPS | JSON | Signature verification |
| Mobile API | Flutter App | Laravel API | HTTPS | JSON | JWT Bearer token |
| Document Upload | Portal | Cloudflare R2 | HTTPS | Binary | Signed URL |
| Notifications | Laravel | Email/SMS | HTTPS | JSON | API Key |

---

## 5. Security Architecture

### 5.1 Security Layers Overview

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    SECURITY ARCHITECTURE                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  LAYER 1: EDGE SECURITY (Cloudflare)                                                               │
│  ════════════════════════════════════                                                               │
│  • WAF (OWASP Core Rule Set)         • Rate Limiting (100 req/min API, 1000 req/min web)          │
│  • DDoS Protection (L3/L4/L7)        • Bot Management                                              │
│  • Geo-blocking (optional)           • IP Reputation Filtering                                     │
│                                                                                                     │
│  LAYER 2: TRANSPORT SECURITY                                                                       │
│  ═══════════════════════════                                                                       │
│  • TLS 1.2+ (All public traffic)     • mTLS (Banking API integration)                             │
│  • Cloudflare Origin Certificates    • Certificate Pinning (Banking)                              │
│  • HSTS Enabled                      • IP Whitelist (Banking)                                     │
│                                                                                                     │
│  LAYER 3: APPLICATION SECURITY                                                                     │
│  ═════════════════════════════                                                                     │
│                                                                                                     │
│  WordPress:                          Laravel:                                                      │
│  • Disable XML-RPC                   • CSRF protection (all forms)                                │
│  • Disable file editing              • XSS prevention (Blade auto-escape)                         │
│  • Hide version info                 • SQL injection prevention (Eloquent)                        │
│  • Limit login attempts              • Input validation (Form Requests)                           │
│  • Two-factor auth (admin)           • Security headers (CSP, X-Frame, etc.)                      │
│  • Security headers                  • Rate limiting (Throttle middleware)                        │
│  • Regular updates                   • Session security (Redis, encrypted)                        │
│  • Minimal plugins                   • Password hashing (Argon2id)                                │
│                                      • MFA for sensitive operations                               │
│                                                                                                     │
│  LAYER 4: DATA SECURITY                                                                            │
│  ══════════════════════                                                                            │
│  • Encryption at Rest (PostgreSQL TDE, Redis, R2, Backups AES-256)                                │
│  • Encryption in Transit (TLS 1.2+ all connections)                                               │
│  • PII Encryption (National ID, KRA PIN, Phone, Bank accounts - app-level)                        │
│  • Key Management (AWS KMS / Vault, 90-day rotation, HSM for banking)                             │
│                                                                                                     │
│  LAYER 5: ACCESS CONTROL                                                                           │
│  ═══════════════════════                                                                           │
│  • Authentication (Email/Phone + Password, MFA, Session management)                               │
│  • Authorization (RBAC, Permission-based policies, Resource ownership)                            │
│  • API Scopes (Mobile app permissions)                                                            │
│                                                                                                     │
│  LAYER 6: AUDIT & COMPLIANCE                                                                       │
│  ═══════════════════════════                                                                       │
│  • All authentication events logged                                                               │
│  • All data modifications logged (before/after values)                                            │
│  • All API access logged                                                                          │
│  • All document access logged                                                                     │
│  • Retention: 7 years (regulatory requirement)                                                    │
│  • Tamper-proof: Append-only, checksummed                                                         │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Banking API Security Protocol

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              CORE BANKING API SECURITY PROTOCOL                                      │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  AUTHENTICATION LAYERS                                                                              │
│  ─────────────────────                                                                              │
│                                                                                                     │
│  1. Network Layer                                                                                   │
│     ┌─────────────────────────────────────────────────────────────────────────────────────────┐   │
│     │  • Dedicated VPN tunnel or private network connection                                   │   │
│     │  • IP whitelist (only EduFin server IPs allowed)                                       │   │
│     │  • Firewall rules restricting source/destination                                       │   │
│     └─────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│  2. Transport Layer (mTLS)                                                                         │
│     ┌─────────────────────────────────────────────────────────────────────────────────────────┐   │
│     │  • Mutual TLS authentication required                                                   │   │
│     │  • Client certificate issued by banking CA                                             │   │
│     │  • Certificate pinning to prevent MITM                                                 │   │
│     │  • TLS 1.3 preferred, TLS 1.2 minimum                                                  │   │
│     │  • Strong cipher suites only                                                           │   │
│     └─────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│  3. Application Layer                                                                              │
│     ┌─────────────────────────────────────────────────────────────────────────────────────────┐   │
│     │  • API Key authentication (X-API-Key header)                                           │   │
│     │  • Request signing (HMAC-SHA256)                                                       │   │
│     │  • Timestamp validation (±5 minute window)                                             │   │
│     │  • Nonce for replay protection                                                         │   │
│     │  • Request/Response encryption (additional layer)                                      │   │
│     └─────────────────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                                     │
│  REQUEST SIGNING ALGORITHM                                                                         │
│  ─────────────────────────                                                                         │
│                                                                                                     │
│  signature = HMAC-SHA256(                                                                          │
│      key = api_secret,                                                                             │
│      message = HTTP_METHOD + "\n" +                                                                │
│                REQUEST_PATH + "\n" +                                                               │
│                TIMESTAMP + "\n" +                                                                  │
│                NONCE + "\n" +                                                                      │
│                SHA256(REQUEST_BODY)                                                                │
│  )                                                                                                 │
│                                                                                                     │
│  HEADERS REQUIRED                                                                                  │
│  ────────────────                                                                                  │
│  X-API-Key: {api_key}                                                                              │
│  X-Timestamp: {unix_timestamp}                                                                     │
│  X-Nonce: {uuid_v4}                                                                                │
│  X-Signature: {base64_encoded_signature}                                                           │
│  Content-Type: application/json                                                                    │
│                                                                                                     │
│  WEBHOOK VERIFICATION                                                                              │
│  ────────────────────                                                                              │
│  • Verify X-Webhook-Signature header                                                               │
│  • Validate timestamp freshness                                                                    │
│  • Check event ID for idempotency                                                                  │
│  • Respond with 200 OK only after successful processing                                            │
│  • Implement retry handling (exponential backoff)                                                  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Role-Based Access Control (RBAC)

### 6.1 Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    ROLE-BASED ACCESS CONTROL                                         │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  INTERNAL ROLES (Staff)                                                                            │
│  ══════════════════════                                                                            │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  SYSTEM ADMINISTRATOR                                                                       │  │
│  │  ─────────────────────                                                                      │  │
│  │  Platform: Laravel (Filament Admin)                                                         │  │
│  │  Permissions:                                                                               │  │
│  │  • Full system access                                                                       │  │
│  │  • User management (create, modify, delete all users)                                      │  │
│  │  • Role assignment                                                                          │  │
│  │  • System configuration                                                                     │  │
│  │  • Audit log access                                                                         │  │
│  │  • API key management                                                                       │  │
│  │  • Database administration                                                                  │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  OPERATIONS MANAGER                                                                         │  │
│  │  ──────────────────                                                                         │  │
│  │  Platform: Laravel (Filament Admin)                                                         │  │
│  │  Permissions:                                                                               │  │
│  │  • View all client accounts                                                                 │  │
│  │  • Approve/reject loan applications                                                         │  │
│  │  • KYC verification decisions                                                               │  │
│  │  • Collateral approval                                                                      │  │
│  │  • Generate reports                                                                         │  │
│  │  • View audit logs (limited)                                                                │  │
│  │  • Cannot modify system settings                                                            │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  LOAN OFFICER                                                                               │  │
│  │  ────────────                                                                               │  │
│  │  Platform: Laravel (Filament Admin)                                                         │  │
│  │  Permissions:                                                                               │  │
│  │  • View assigned client accounts                                                            │  │
│  │  • Process loan applications                                                                │  │
│  │  • Request additional documents                                                             │  │
│  │  • Add notes to applications                                                                │  │
│  │  • Escalate to Operations Manager                                                           │  │
│  │  • Cannot approve loans (recommend only)                                                    │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  SECRETARIAL STAFF                                                                          │  │
│  │  ─────────────────                                                                          │  │
│  │  Platform: WordPress (Admin Dashboard)                                                      │  │
│  │  Permissions:                                                                               │  │
│  │  • Manage blog posts and pages                                                              │  │
│  │  • Manage FAQs and testimonials                                                             │  │
│  │  • View contact form submissions                                                            │  │
│  │  • Manage newsletter subscribers                                                            │  │
│  │  • Upload media files                                                                       │  │
│  │  • Cannot access Laravel portal                                                             │  │
│  │  • Cannot modify plugins or settings                                                        │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  EXTERNAL ROLES (Clients)                                                                          │
│  ════════════════════════                                                                          │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  CLIENT (Primary Applicant / Account Holder)                                                │  │
│  │  ───────────────────────────────────────────                                                │  │
│  │  Platform: Laravel (Client Portal) + Mobile App                                             │  │
│  │  Permissions:                                                                               │  │
│  │  • Complete own profile and KYC                                                             │  │
│  │  • Add/manage education beneficiaries (dependents)                                          │  │
│  │  • Apply for loans                                                                          │  │
│  │  • Register collateral                                                                      │  │
│  │  • View loan status and history                                                             │  │
│  │  • View statements and payment history                                                      │  │
│  │  • Upload documents                                                                         │  │
│  │  • Update contact information                                                               │  │
│  │  • Manage notification preferences                                                          │  │
│  │  • Grant portal access to dependents                                                        │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  DEPENDENT (Education Beneficiary with Portal Access)                                       │  │
│  │  ────────────────────────────────────────────────                                           │  │
│  │  Platform: Laravel (Client Portal) + Mobile App                                             │  │
│  │  Permissions:                                                                               │  │
│  │  • View own profile                                                                         │  │
│  │  • View loan status (related to their education)                                            │  │
│  │  • View payment schedule                                                                    │  │
│  │  • Upload school documents (fee structures, results)                                        │  │
│  │  • Update own contact information                                                           │  │
│  │  • Cannot apply for loans                                                                   │  │
│  │  • Cannot view other beneficiaries                                                          │  │
│  │  • Cannot modify financial information                                                      │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Permission Matrix

| Permission | Sys Admin | Ops Manager | Loan Officer | Secretary | Client | Dependent |
|------------|:---------:|:-----------:|:------------:|:---------:|:------:|:---------:|
| **System Configuration** | ✓ | - | - | - | - | - |
| **User Management** | ✓ | - | - | - | - | - |
| **Role Assignment** | ✓ | - | - | - | - | - |
| **View All Clients** | ✓ | ✓ | - | - | - | - |
| **View Assigned Clients** | ✓ | ✓ | ✓ | - | - | - |
| **Approve Loans** | ✓ | ✓ | - | - | - | - |
| **Process Loans** | ✓ | ✓ | ✓ | - | - | - |
| **KYC Verification** | ✓ | ✓ | ✓ | - | - | - |
| **View Audit Logs** | ✓ | ✓ | - | - | - | - |
| **Manage WP Content** | - | - | - | ✓ | - | - |
| **Apply for Loan** | - | - | - | - | ✓ | - |
| **View Own Account** | - | - | - | - | ✓ | ✓ |
| **Manage Dependents** | - | - | - | - | ✓ | - |
| **Upload Documents** | - | - | - | - | ✓ | ✓ |

### 6.3 Laravel Permission Implementation

```php
// app/Models/Permission.php - Permission Categories

return [
    // System Administration
    'system.config.view',
    'system.config.edit',
    'system.users.view',
    'system.users.create',
    'system.users.edit',
    'system.users.delete',
    'system.roles.manage',
    'system.audit.view',
    
    // Client Management
    'clients.view.all',
    'clients.view.assigned',
    'clients.create',
    'clients.edit',
    'clients.delete',
    
    // Loan Management
    'loans.view.all',
    'loans.view.assigned',
    'loans.process',
    'loans.approve',
    'loans.reject',
    'loans.disburse',
    
    // KYC Management
    'kyc.view',
    'kyc.verify',
    'kyc.reject',
    
    // Collateral Management
    'collateral.view',
    'collateral.approve',
    'collateral.reject',
    
    // Reports
    'reports.view',
    'reports.export',
    
    // Client Portal (External)
    'portal.profile.view',
    'portal.profile.edit',
    'portal.beneficiaries.manage',
    'portal.loans.apply',
    'portal.loans.view',
    'portal.documents.upload',
    'portal.statements.view',
];
```

---

## 7. API Architecture & Banking Integration

### 7.1 API Layer Structure

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    API ARCHITECTURE                                                  │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  PUBLIC API (api.edufin.co.ke/api/v1)                                                              │
│  ════════════════════════════════════                                                              │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  UNAUTHENTICATED ENDPOINTS (WordPress Integration)                                          │  │
│  │  ──────────────────────────────────────────────────                                         │  │
│  │                                                                                             │  │
│  │  GET  /api/v1/packages              # List financing packages (cached)                     │  │
│  │  GET  /api/v1/packages/{slug}       # Package details                                      │  │
│  │  GET  /api/v1/calculator/rates      # Loan calculator rates                                │  │
│  │  POST /api/v1/inquiries             # Submit contact inquiry                               │  │
│  │  POST /api/v1/newsletter/subscribe  # Newsletter signup                                    │  │
│  │                                                                                             │  │
│  │  Security: API Key (X-API-Key header)                                                      │  │
│  │  Rate Limit: 100 requests/minute                                                           │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  SSO ENDPOINTS (WordPress ↔ Laravel)                                                        │  │
│  │  ───────────────────────────────────                                                        │  │
│  │                                                                                             │  │
│  │  GET  /sso/login                    # Initiate SSO login                                   │  │
│  │  POST /api/v1/sso/validate          # Validate one-time token                              │  │
│  │  POST /api/v1/sso/session           # Validate persistent session                          │  │
│  │  POST /sso/logout                   # Logout and invalidate sessions                       │  │
│  │  GET  /api/v1/sso/me                # Get current user info                                │  │
│  │                                                                                             │  │
│  │  Security: API Key + One-time tokens + Session cookies                                     │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  AUTHENTICATED ENDPOINTS (Mobile App / Portal)                                              │  │
│  │  ─────────────────────────────────────────────                                              │  │
│  │                                                                                             │  │
│  │  Authentication                                                                             │  │
│  │  POST /api/v1/auth/register         # New user registration                                │  │
│  │  POST /api/v1/auth/login            # Login (returns JWT)                                  │  │
│  │  POST /api/v1/auth/refresh          # Refresh JWT token                                    │  │
│  │  POST /api/v1/auth/logout           # Logout                                               │  │
│  │  POST /api/v1/auth/verify-otp       # Verify phone/email OTP                               │  │
│  │  POST /api/v1/auth/forgot-password  # Password reset request                               │  │
│  │                                                                                             │  │
│  │  Profile                                                                                    │  │
│  │  GET  /api/v1/profile               # Get user profile                                     │  │
│  │  PUT  /api/v1/profile               # Update profile                                       │  │
│  │  POST /api/v1/profile/kyc           # Submit KYC documents                                 │  │
│  │  GET  /api/v1/profile/kyc/status    # KYC verification status                              │  │
│  │                                                                                             │  │
│  │  Beneficiaries                                                                              │  │
│  │  GET  /api/v1/beneficiaries         # List beneficiaries                                   │  │
│  │  POST /api/v1/beneficiaries         # Add beneficiary                                      │  │
│  │  GET  /api/v1/beneficiaries/{id}    # Beneficiary details                                  │  │
│  │  PUT  /api/v1/beneficiaries/{id}    # Update beneficiary                                   │  │
│  │                                                                                             │  │
│  │  Loans                                                                                      │  │
│  │  GET  /api/v1/loans                 # List user's loans                                    │  │
│  │  POST /api/v1/loans                 # Apply for loan                                       │  │
│  │  GET  /api/v1/loans/{id}            # Loan details                                         │  │
│  │  GET  /api/v1/loans/{id}/schedule   # Payment schedule                                     │  │
│  │  GET  /api/v1/loans/{id}/statement  # Generate statement                                   │  │
│  │                                                                                             │  │
│  │  Collateral                                                                                 │  │
│  │  GET  /api/v1/collateral            # List collateral                                      │  │
│  │  POST /api/v1/collateral            # Register collateral                                  │  │
│  │  GET  /api/v1/collateral/{id}       # Collateral details                                   │  │
│  │                                                                                             │  │
│  │  Documents                                                                                  │  │
│  │  POST /api/v1/documents/upload      # Upload document (returns signed URL)                 │  │
│  │  GET  /api/v1/documents             # List documents                                       │  │
│  │  GET  /api/v1/documents/{id}        # Download document (signed URL)                       │  │
│  │                                                                                             │  │
│  │  Notifications                                                                              │  │
│  │  GET  /api/v1/notifications         # List notifications                                   │  │
│  │  PUT  /api/v1/notifications/{id}    # Mark as read                                         │  │
│  │  PUT  /api/v1/notifications/read-all # Mark all as read                                    │  │
│  │                                                                                             │  │
│  │  Security: JWT Bearer Token                                                                │  │
│  │  Rate Limit: 60 requests/minute per user                                                   │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │  WEBHOOK ENDPOINTS (External Systems)                                                       │  │
│  │  ────────────────────────────────────                                                       │  │
│  │                                                                                             │  │
│  │  POST /api/v1/webhooks/banking/payment    # Payment notification from CBS                  │  │
│  │  POST /api/v1/webhooks/banking/status     # Loan status update from CBS                    │  │
│  │  POST /api/v1/webhooks/mpesa/callback     # M-Pesa payment callback                        │  │
│  │  POST /api/v1/webhooks/sms/delivery       # SMS delivery report                            │  │
│  │                                                                                             │  │
│  │  Security: Signature verification + IP whitelist                                           │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Core Banking Integration

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              CORE BANKING SYSTEM INTEGRATION                                         │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  INTEGRATION PATTERN: Asynchronous with Synchronous Fallback                                       │
│  ═══════════════════════════════════════════════════════════                                       │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  EDUFIN LARAVEL                           CORE BANKING SYSTEM                              │  │
│  │  ─────────────────                        ───────────────────                              │  │
│  │                                                                                             │  │
│  │  ┌───────────────┐                        ┌───────────────┐                               │  │
│  │  │   Loan        │   1. Loan Approved     │   CBS API     │                               │  │
│  │  │   Service     │ ─────────────────────► │   Gateway     │                               │  │
│  │  │               │   POST /loans          │               │                               │  │
│  │  │               │   (mTLS + Signed)      │               │                               │  │
│  │  └───────────────┘                        └───────┬───────┘                               │  │
│  │         │                                         │                                        │  │
│  │         │                                         │ 2. Process                             │  │
│  │         │                                         ▼                                        │  │
│  │         │                                 ┌───────────────┐                               │  │
│  │         │                                 │   Core        │                               │  │
│  │         │                                 │   Banking     │                               │  │
│  │         │                                 │   Engine      │                               │  │
│  │         │                                 └───────┬───────┘                               │  │
│  │         │                                         │                                        │  │
│  │         │   4. Update Status                      │ 3. Webhook                             │  │
│  │         ▼                                         ▼                                        │  │
│  │  ┌───────────────┐                        ┌───────────────┐                               │  │
│  │  │   PostgreSQL  │ ◄───────────────────── │   Webhook     │                               │  │
│  │  │   (Loan       │   POST /webhooks/      │   Dispatcher  │                               │  │
│  │  │    Status)    │   banking/status       │               │                               │  │
│  │  └───────────────┘                        └───────────────┘                               │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  OPERATIONS SUPPORTED                                                                              │
│  ════════════════════                                                                              │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  OUTBOUND (EduFin → CBS)                  INBOUND (CBS → EduFin)                           │  │
│  │  ───────────────────────                  ─────────────────────                            │  │
│  │                                                                                             │  │
│  │  • Create Loan Account                    • Loan Status Update                             │  │
│  │  • Request Disbursement                   • Disbursement Confirmation                      │  │
│  │  • Update Loan Terms                      • Payment Received                               │  │
│  │  • Request Balance                        • Payment Reversal                               │  │
│  │  • Request Statement                      • Interest Accrual                               │  │
│  │  • Register Collateral Lien               • Penalty Applied                                │  │
│  │  • Release Collateral Lien                • Account Closure                                │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ERROR HANDLING & RESILIENCE                                                                       │
│  ═══════════════════════════                                                                       │
│                                                                                                     │
│  • Circuit Breaker: Opens after 5 consecutive failures, half-open after 30 seconds                │
│  • Retry Policy: 3 retries with exponential backoff (1s, 2s, 4s)                                  │
│  • Timeout: 30 seconds per request                                                                │
│  • Fallback: Queue for manual processing if CBS unavailable                                       │
│  • Idempotency: All requests include idempotency key                                              │
│  • Reconciliation: Daily batch reconciliation job                                                 │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Separation of Concerns Analysis

### 8.1 Benefits of Dual-Platform Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              SEPARATION OF CONCERNS BENEFITS                                         │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  SEO & MARKETING OPTIMIZATION (WordPress)                                                          │
│  ════════════════════════════════════════                                                          │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  WHY WORDPRESS FOR MARKETING:                                                              │  │
│  │                                                                                             │  │
│  │  1. SEO Excellence                                                                         │  │
│  │     • Native SEO-friendly URL structures                                                   │  │
│  │     • Rich plugin ecosystem (Yoast, Rank Math)                                            │  │
│  │     • Schema markup support                                                                │  │
│  │     • XML sitemaps auto-generation                                                         │  │
│  │     • Fast page load (with caching plugins)                                               │  │
│  │     • Mobile-responsive themes                                                             │  │
│  │                                                                                             │  │
│  │  2. Content Management Efficiency                                                          │  │
│  │     • Non-technical staff can manage content                                              │  │
│  │     • WYSIWYG editor for blog posts                                                       │  │
│  │     • Media library management                                                             │  │
│  │     • Scheduled publishing                                                                 │  │
│  │     • Revision history                                                                     │  │
│  │                                                                                             │  │
│  │  3. Marketing Integration                                                                  │  │
│  │     • Newsletter plugins (Mailchimp, ConvertKit)                                          │  │
│  │     • Social media integration                                                             │  │
│  │     • Analytics integration (GA4, GTM)                                                    │  │
│  │     • A/B testing capabilities                                                             │  │
│  │     • Landing page builders                                                                │  │
│  │                                                                                             │  │
│  │  4. Cost Efficiency                                                                        │  │
│  │     • Lower hosting requirements                                                           │  │
│  │     • Abundant developer talent                                                            │  │
│  │     • Extensive theme/plugin marketplace                                                   │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  SECURITY & SCALABILITY (Laravel)                                                                  │
│  ════════════════════════════════                                                                  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  WHY LARAVEL FOR TRANSACTIONS:                                                             │  │
│  │                                                                                             │  │
│  │  1. Security Hardening                                                                     │  │
│  │     • Built-in CSRF protection                                                             │  │
│  │     • SQL injection prevention (Eloquent ORM)                                             │  │
│  │     • XSS prevention (Blade templating)                                                   │  │
│  │     • Encryption services (AES-256)                                                       │  │
│  │     • Secure session management                                                            │  │
│  │     • Rate limiting middleware                                                             │  │
│  │     • No plugin vulnerabilities                                                            │  │
│  │                                                                                             │  │
│  │  2. Financial-Grade Architecture                                                           │  │
│  │     • ACID-compliant transactions                                                          │  │
│  │     • Audit logging built-in                                                               │  │
│  │     • Role-based access control                                                            │  │
│  │     • API versioning support                                                               │  │
│  │     • Queue system for async processing                                                    │  │
│  │     • Event-driven architecture                                                            │  │
│  │                                                                                             │  │
│  │  3. Scalability                                                                            │  │
│  │     • Horizontal scaling (stateless)                                                       │  │
│  │     • Redis caching                                                                        │  │
│  │     • Queue workers (Horizon)                                                              │  │
│  │     • Database connection pooling                                                          │  │
│  │     • Load balancer ready                                                                  │  │
│  │                                                                                             │  │
│  │  4. API-First Design                                                                       │  │
│  │     • RESTful API for mobile apps                                                          │  │
│  │     • JWT authentication                                                                   │  │
│  │     • API rate limiting                                                                    │  │
│  │     • Versioned endpoints                                                                  │  │
│  │     • OpenAPI documentation                                                                │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ISOLATION BENEFITS                                                                                │
│  ═════════════════                                                                                 │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  SECURITY ISOLATION                       OPERATIONAL ISOLATION                            │  │
│  │  ──────────────────                       ─────────────────────                            │  │
│  │                                                                                             │  │
│  │  • WordPress compromise does NOT         • WordPress updates don't affect                 │  │
│  │    expose financial data                   transaction processing                          │  │
│  │                                                                                             │  │
│  │  • WordPress has NO access to            • Laravel can be scaled independently            │  │
│  │    Laravel database                        based on transaction load                       │  │
│  │                                                                                             │  │
│  │  • Different security postures           • Different deployment cycles                    │  │
│  │    for different risk levels               (content vs. code)                             │  │
│  │                                                                                             │  │
│  │  • PII never touches WordPress           • Marketing team can work without               │  │
│  │                                            affecting core systems                          │  │
│  │                                                                                             │  │
│  │  COMPLIANCE BENEFITS                      DEVELOPMENT BENEFITS                             │  │
│  │  ──────────────────                       ────────────────────                             │  │
│  │                                                                                             │  │
│  │  • Clear data boundaries for             • Specialized teams for each                     │  │
│  │    regulatory audits                       platform                                        │  │
│  │                                                                                             │  │
│  │  • Easier to demonstrate                 • Different testing strategies                   │  │
│  │    security controls                                                                       │  │
│  │                                                                                             │  │
│  │  • Simplified PCI-DSS scope              • Parallel development possible                  │  │
│  │    (if applicable)                                                                         │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Comparison: Unified vs. Dual-Platform

| Aspect | Unified Laravel | Dual-Platform (WP + Laravel) |
|--------|-----------------|------------------------------|
| **SEO Capability** | Good (with effort) | Excellent (native) |
| **Content Management** | Requires custom CMS | Native WordPress |
| **Security Surface** | Single attack surface | Isolated surfaces |
| **Maintenance Overhead** | Lower | Higher (2 systems) |
| **Team Requirements** | Laravel developers | Laravel + WordPress |
| **Scalability** | Unified scaling | Independent scaling |
| **Plugin Vulnerabilities** | None | WordPress plugins |
| **Data Isolation** | Application-level | System-level |
| **Regulatory Compliance** | Harder boundaries | Clear boundaries |
| **Marketing Agility** | Slower (dev needed) | Faster (non-technical) |

---

## 9. Mobile Ecosystem Transition Strategy

### 9.1 Mobile Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              MOBILE ECOSYSTEM ARCHITECTURE                                           │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  FLUTTER APPLICATION ARCHITECTURE                                                                  │
│  ════════════════════════════════                                                                  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │                              FLUTTER APP (iOS & Android)                                   │  │
│  │                                                                                             │  │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐  │  │
│  │  │                            PRESENTATION LAYER                                       │  │  │
│  │  │                                                                                     │  │  │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │  │  │
│  │  │  │   Login     │  │  Dashboard  │  │   Loans     │  │  Profile    │              │  │  │
│  │  │  │   Screen    │  │   Screen    │  │   Screen    │  │   Screen    │              │  │  │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘              │  │  │
│  │  │                                                                                     │  │  │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │  │  │
│  │  │  │Beneficiaries│  │  Documents  │  │ Statements  │  │Notifications│              │  │  │
│  │  │  │   Screen    │  │   Screen    │  │   Screen    │  │   Screen    │              │  │  │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘              │  │  │
│  │  │                                                                                     │  │  │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘  │  │
│  │                                           │                                               │  │
│  │                                           ▼                                               │  │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐  │  │
│  │  │                            BUSINESS LOGIC LAYER (BLoC)                              │  │  │
│  │  │                                                                                     │  │  │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │  │  │
│  │  │  │   Auth      │  │   Loan      │  │Beneficiary  │  │  Document   │              │  │  │
│  │  │  │   BLoC      │  │   BLoC      │  │   BLoC      │  │   BLoC      │              │  │  │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘              │  │  │
│  │  │                                                                                     │  │  │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘  │  │
│  │                                           │                                               │  │
│  │                                           ▼                                               │  │
│  │  ┌─────────────────────────────────────────────────────────────────────────────────────┐  │  │
│  │  │                            DATA LAYER                                               │  │  │
│  │  │                                                                                     │  │  │
│  │  │  ┌─────────────────────────────┐  ┌─────────────────────────────┐                 │  │  │
│  │  │  │       Repositories          │  │       Data Sources          │                 │  │  │
│  │  │  │                             │  │                             │                 │  │  │
│  │  │  │  • AuthRepository           │  │  • ApiDataSource            │                 │  │  │
│  │  │  │  • LoanRepository           │  │    (api.edufin.co.ke)       │                 │  │  │
│  │  │  │  • BeneficiaryRepository    │  │                             │                 │  │  │
│  │  │  │  • DocumentRepository       │  │  • LocalDataSource          │                 │  │  │
│  │  │  │                             │  │    (SQLite/Hive)            │                 │  │  │
│  │  │  └─────────────────────────────┘  └─────────────────────────────┘                 │  │  │
│  │  │                                                                                     │  │  │
│  │  └─────────────────────────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                           │                                                       │
│                                           │ HTTPS + JWT                                           │
│                                           ▼                                                       │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │                              LARAVEL API (api.edufin.co.ke)                                │  │
│  │                                                                                             │  │
│  │  Same API endpoints used by web portal - no additional backend development required        │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 9.2 API-First Design Benefits for Mobile

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              API-FIRST DESIGN FOR MOBILE TRANSITION                                  │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  SHARED API LAYER                                                                                  │
│  ════════════════                                                                                  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │                              LARAVEL API (/api/v1/*)                                       │  │
│  │                                                                                             │  │
│  │                    ┌───────────────────────────────────────┐                              │  │
│  │                    │                                       │                              │  │
│  │     ┌──────────────┼──────────────┬──────────────┬────────┼──────────────┐               │  │
│  │     │              │              │              │        │              │               │  │
│  │     ▼              ▼              ▼              ▼        ▼              ▼               │  │
│  │  ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐    ┌──────┐     ┌──────┐              │  │
│  │  │ Web  │     │ iOS  │     │Android│    │ Admin│    │ WP   │     │Future│              │  │
│  │  │Portal│     │ App  │     │ App  │     │Panel │    │ Site │     │ Apps │              │  │
│  │  │(Live │     │(Flutt│     │(Flutt│     │(Fila │    │(API  │     │      │              │  │
│  │  │wire) │     │er)   │     │er)   │     │ment) │    │Client│     │      │              │  │
│  │  └──────┘     └──────┘     └──────┘     └──────┘    └──────┘     └──────┘              │  │
│  │                                                                                             │  │
│  │  ALL CLIENTS USE THE SAME API ENDPOINTS                                                    │  │
│  │  • Consistent business logic                                                               │  │
│  │  • Single source of truth                                                                  │  │
│  │  • Unified validation rules                                                                │  │
│  │  • Centralized security                                                                    │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  MOBILE-SPECIFIC CONSIDERATIONS                                                                    │
│  ══════════════════════════════                                                                    │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  1. Authentication                                                                         │  │
│  │     • JWT tokens (access + refresh)                                                        │  │
│  │     • Biometric authentication support                                                     │  │
│  │     • Device registration & fingerprinting                                                 │  │
│  │     • Push notification tokens                                                             │  │
│  │                                                                                             │  │
│  │  2. Offline Support                                                                        │  │
│  │     • Local caching of frequently accessed data                                           │  │
│  │     • Offline queue for actions (sync when online)                                        │  │
│  │     • Conflict resolution strategy                                                         │  │
│  │                                                                                             │  │
│  │  3. Push Notifications                                                                     │  │
│  │     • Firebase Cloud Messaging (FCM)                                                       │  │
│  │     • Apple Push Notification Service (APNs)                                              │  │
│  │     • Notification preferences API                                                         │  │
│  │                                                                                             │  │
│  │  4. Security                                                                               │  │
│  │     • Certificate pinning                                                                  │  │
│  │     • Secure storage (Keychain/Keystore)                                                  │  │
│  │     • Root/Jailbreak detection                                                             │  │
│  │     • App integrity verification                                                           │  │
│  │                                                                                             │  │
│  │  5. Performance                                                                            │  │
│  │     • Pagination for list endpoints                                                        │  │
│  │     • Compressed responses (gzip)                                                          │  │
│  │     • Image optimization (WebP, thumbnails)                                               │  │
│  │     • Delta sync for large datasets                                                        │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  MOBILE DEVELOPMENT TIMELINE                                                                       │
│  ═══════════════════════════                                                                       │
│                                                                                                     │
│  Phase 1 (Web Portal Launch):     API already built for Livewire portal                           │
│  Phase 2 (Mobile Development):    Flutter app consumes existing API                               │
│  Phase 3 (Mobile Launch):         iOS & Android apps released                                     │
│  Phase 4 (Feature Parity):        All web features available on mobile                            │
│                                                                                                     │
│  ESTIMATED MOBILE DEVELOPMENT EFFORT: 60% less than building from scratch                         │
│  (API layer already exists, only UI/UX development needed)                                        │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 10. Infrastructure Architecture

### 10.1 Production Infrastructure

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              PRODUCTION INFRASTRUCTURE                                               │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  RECOMMENDED HOSTING: DigitalOcean / AWS / Hetzner                                                 │
│  ═════════════════════════════════════════════════                                                 │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  WORDPRESS SERVER                         LARAVEL SERVER(S)                                │  │
│  │  ─────────────────                        ─────────────────                                │  │
│  │                                                                                             │  │
│  │  Droplet/Instance: 2 vCPU, 4GB RAM       Droplet/Instance: 4 vCPU, 8GB RAM               │  │
│  │  OS: Ubuntu 22.04 LTS                    OS: Ubuntu 22.04 LTS                             │  │
│  │  Storage: 80GB SSD                       Storage: 160GB SSD                               │  │
│  │                                                                                             │  │
│  │  Software:                               Software:                                         │  │
│  │  • Nginx 1.24                            • Nginx 1.24                                     │  │
│  │  • PHP 8.2-FPM                           • PHP 8.3-FPM                                    │  │
│  │  • MySQL 8.0                             • Supervisor (Horizon)                           │  │
│  │  • Redis 7 (Object Cache)                                                                 │  │
│  │                                                                                             │  │
│  │  Est. Cost: $24/month                    Est. Cost: $48/month (single)                    │  │
│  │                                          Est. Cost: $96/month (HA pair)                   │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  MANAGED DATABASES                        SHARED SERVICES                                  │  │
│  │  ─────────────────                        ───────────────                                  │  │
│  │                                                                                             │  │
│  │  PostgreSQL (Managed)                    Redis (Managed)                                  │  │
│  │  • DigitalOcean Managed DB               • Upstash / Redis Cloud                          │  │
│  │  • 2 vCPU, 4GB RAM                       • 256MB (Sessions, Cache)                        │  │
│  │  • 50GB Storage                          • Est. Cost: $10/month                           │  │
│  │  • Daily backups                                                                           │  │
│  │  • Est. Cost: $60/month                  Cloudflare R2 (Storage)                          │  │
│  │                                          • 10GB included free                             │  │
│  │  MySQL (Managed) - WordPress             • $0.015/GB after                                │  │
│  │  • DigitalOcean Managed DB               • Est. Cost: $5/month                            │  │
│  │  • 1 vCPU, 2GB RAM                                                                        │  │
│  │  • 25GB Storage                          Backblaze B2 (Backups)                           │  │
│  │  • Est. Cost: $30/month                  • $0.005/GB storage                              │  │
│  │                                          • Est. Cost: $5/month                            │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
│  ESTIMATED MONTHLY COSTS                                                                           │
│  ═══════════════════════                                                                           │
│                                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                                             │  │
│  │  Component                    MVP Setup              Production Setup                      │  │
│  │  ─────────────────────────────────────────────────────────────────────                     │  │
│  │  WordPress Server             $24                    $24                                   │  │
│  │  Laravel Server               $48                    $96 (2 servers)                       │  │
│  │  PostgreSQL (Managed)         $60                    $120 (HA)                             │  │
│  │  MySQL (Managed)              $30                    $30                                   │  │
│  │  Redis (Managed)              $10                    $20                                   │  │
│  │  Cloudflare (Pro)             $20                    $20                                   │  │
│  │  R2 Storage                   $5                     $10                                   │  │
│  │  Backblaze B2                 $5                     $10                                   │  │
│  │  Monitoring (Uptime Robot)    $7                     $7                                    │  │
│  │  Error Tracking (Sentry)      $0 (free tier)         $26                                   │  │
│  │  ─────────────────────────────────────────────────────────────────────                     │  │
│  │  TOTAL                        ~$209/month            ~$363/month                           │  │
│  │                                                                                             │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 10.2 High Availability Architecture (Future)

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              HIGH AVAILABILITY ARCHITECTURE                                          │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│                                      CLOUDFLARE                                                     │
│                                          │                                                          │
│                    ┌─────────────────────┼─────────────────────┐                                   │
│                    │                     │                     │                                    │
│                    ▼                     ▼                     ▼                                    │
│             ┌───────────┐         ┌───────────┐         ┌───────────┐                             │
│             │ WordPress │         │  Laravel  │         │  Laravel  │                             │
│             │  Server   │         │ Server 1  │         │ Server 2  │                             │
│             │           │         │ (Primary) │         │(Secondary)│                             │
│             └─────┬─────┘         └─────┬─────┘         └─────┬─────┘                             │
│                   │                     │                     │                                    │
│                   │                     └──────────┬──────────┘                                    │
│                   │                                │                                               │
│                   ▼                                ▼                                               │
│             ┌───────────┐               ┌───────────────────┐                                     │
│             │  MySQL    │               │    PostgreSQL     │                                     │
│             │ (Managed) │               │   (HA Cluster)    │                                     │
│             │           │               │                   │                                     │
│             │  Primary  │               │ Primary + Standby │                                     │
│             └───────────┘               └───────────────────┘                                     │
│                                                                                                     │
│                                    ┌───────────────────┐                                           │
│                                    │   Redis Cluster   │                                           │
│                                    │   (HA Mode)       │                                           │
│                                    └───────────────────┘                                           │
│                                                                                                     │
│  FAILOVER STRATEGY                                                                                 │
│  ─────────────────                                                                                 │
│  • Cloudflare handles load balancing and failover                                                 │
│  • PostgreSQL automatic failover (managed service)                                                │
│  • Redis Sentinel for cache failover                                                              │
│  • Laravel servers are stateless (any can handle requests)                                        │
│  • Session data in Redis (shared across servers)                                                  │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 11. Integration Points & Contracts

### 11.1 WordPress ↔ Laravel Integration

| Integration Point | Direction | Protocol | Authentication | Data Format |
|-------------------|-----------|----------|----------------|-------------|
| SSO Login | WP → Laravel | HTTPS | Redirect + Token | URL params |
| SSO Validate | WP → Laravel | HTTPS POST | API Key | JSON |
| SSO Session | WP → Laravel | HTTPS POST | API Key | JSON |
| Get Packages | WP → Laravel | HTTPS GET | API Key | JSON |
| Get Rates | WP → Laravel | HTTPS GET | API Key | JSON |
| Submit Inquiry | WP → Laravel | HTTPS POST | API Key | JSON |
| Newsletter Sync | WP → Laravel | HTTPS POST | API Key | JSON |

### 11.2 Laravel ↔ Core Banking Integration

| Integration Point | Direction | Protocol | Authentication | Data Format |
|-------------------|-----------|----------|----------------|-------------|
| Create Loan | Laravel → CBS | HTTPS + mTLS | Cert + API Key + Signature | JSON |
| Disbursement | Laravel → CBS | HTTPS + mTLS | Cert + API Key + Signature | JSON |
| Balance Inquiry | Laravel → CBS | HTTPS + mTLS | Cert + API Key + Signature | JSON |
| Payment Webhook | CBS → Laravel | HTTPS | Signature Verification | JSON |
| Status Webhook | CBS → Laravel | HTTPS | Signature Verification | JSON |

### 11.3 Laravel ↔ External Services

| Service | Purpose | Protocol | Authentication |
|---------|---------|----------|----------------|
| Africa's Talking | SMS Notifications | HTTPS | API Key |
| SendGrid | Email Notifications | HTTPS | API Key |
| M-Pesa | Payment Processing | HTTPS | OAuth + API Key |
| NTSA | Vehicle Verification | HTTPS | API Key |
| Cloudflare R2 | Document Storage | HTTPS | Access Key + Secret |

---

## 12. Scalability & Performance

### 12.1 Scaling Strategy

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                              SCALING STRATEGY                                                        │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                     │
│  PHASE 1: VERTICAL SCALING (0-1000 users)                                                          │
│  ════════════════════════════════════════                                                          │
│  • Upgrade server resources as needed                                                              │
│  • Single Laravel server handles all traffic                                                       │
│  • Managed database with automatic scaling                                                         │
│                                                                                                     │
│  PHASE 2: HORIZONTAL SCALING (1000-10000 users)                                                    │
│  ═════════════════════════════════════════════                                                     │
│  • Add second Laravel server behind load balancer                                                  │
│  • Separate queue workers to dedicated server                                                      │
│  • Redis cluster for sessions and cache                                                            │
│  • Database read replicas                                                                          │
│                                                                                                     │
│  PHASE 3: MICROSERVICES (10000+ users)                                                             │
│  ═════════════════════════════════════                                                             │
│  • Extract high-load services (notifications, documents)                                           │
│  • Kubernetes orchestration                                                                        │
│  • Auto-scaling based on load                                                                      │
│  • Multi-region deployment                                                                         │
│                                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 12.2 Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Page Load Time (WordPress) | < 2 seconds | Google PageSpeed |
| API Response Time (p95) | < 500ms | Application monitoring |
| API Response Time (p99) | < 1000ms | Application monitoring |
| Uptime | 99.9% | Uptime monitoring |
| Error Rate | < 0.1% | Error tracking |
| Database Query Time (p95) | < 100ms | Query monitoring |

### 12.3 Caching Strategy

| Layer | Technology | TTL | Purpose |
|-------|------------|-----|---------|
| Edge | Cloudflare CDN | 1 hour | Static assets, images |
| Page | WP Rocket | 1 hour | WordPress pages |
| API | Redis | 5-60 min | API responses |
| Database | Redis | 15 min | Query results |
| Session | Redis | 24 hours | User sessions |
| Object | Redis | Varies | Application objects |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Team | Initial technical architecture |

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **CBS** | Core Banking System |
| **mTLS** | Mutual TLS (two-way certificate authentication) |
| **RBAC** | Role-Based Access Control |
| **SSO** | Single Sign-On |
| **JWT** | JSON Web Token |
| **PII** | Personally Identifiable Information |
| **KYC** | Know Your Customer |
| **HMAC** | Hash-based Message Authentication Code |

---

*This document provides the comprehensive technical architecture for the EduFin dual-platform ecosystem. It should be used as the authoritative reference for all development, deployment, and integration decisions.*
