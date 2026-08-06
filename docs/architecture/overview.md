# EduFin - Architecture Overview

## System Architecture

**Version:** 1.0  
**Last Updated:** August 6, 2026

---

## Dual-Platform Architecture

EduFin operates as a dual-platform ecosystem with strict separation of concerns:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           EDUFIN ARCHITECTURE                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                              CLOUDFLARE                                         │
│                    (DNS, CDN, WAF, SSL Termination)                            │
│                                   │                                             │
│              ┌────────────────────┴────────────────────┐                       │
│              │                                          │                        │
│              ▼                                          ▼                        │
│     ┌─────────────────┐                    ┌─────────────────┐                 │
│     │ edufin.co.ke    │                    │app.edufin.co.ke │                 │
│     │   WORDPRESS     │   /api/v1 (proxy)  │    LARAVEL      │                 │
│     │  Landing Page   │───────────────────►│  Client Portal  │                 │
│     │                 │                    │  Admin (Filament)│                 │
│     └────────┬────────┘                    └────────┬────────┘                 │
│              │                                      │                          │
│              ▼                                      ▼                          │
│     ┌─────────────────┐         ┌─────────────────────────────────┐           │
│     │     MySQL       │         │         PostgreSQL              │           │
│     │  (Content Only) │         │    (All Business Data)          │           │
│     └─────────────────┘         └─────────────────────────────────┘           │
│                                              │                                  │
│                                              ▼                                  │
│                                 ┌─────────────────────────────────┐           │
│                                 │      CORE BANKING SYSTEM        │           │
│                                 │        (External API)           │           │
│                                 └─────────────────────────────────┘           │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Platform Responsibilities

### WordPress (Landing Page)
- **Purpose:** Public-facing brand presence
- **Managed by:** Secretarial staff
- **Contains:** Marketing content, blog, FAQs
- **Does NOT contain:** Business data, PII, financial information

### Laravel (Core Application)
- **Purpose:** All business operations
- **Domain:** `app.edufin.co.ke` (portal + admin) and `edufin.co.ke/api/v1` (REST API via Nginx path routing on the main domain)
- **Components:**
  - Client Portal (Livewire) — `app.edufin.co.ke/dashboard`
  - Admin Panel (Filament) — `app.edufin.co.ke/admin`
  - REST API (Sanctum) — `edufin.co.ke/api/v1`
  - Queue Workers (Horizon)
- **Authentication:** Login at `app.edufin.co.ke/login`; onboarding at `app.edufin.co.ke/register`. After login, users are redirected by role:
  - Client → `app.edufin.co.ke/dashboard`
  - Staff (Loan Officer, KYC Verifier, System Admin, Super Admin) → `app.edufin.co.ke/admin`
- **Contains:** Users, loans, KYC, documents, transactions

## Integration Points

| Integration | Protocol | Purpose |
|-------------|----------|---------|
| Laravel → CBS | HTTPS + mTLS | Financial operations |
| Mobile → Laravel API (`edufin.co.ke/api/v1`) | HTTPS + JWT | Client access |
| CBS → Laravel API (`edufin.co.ke/api/v1`) | HTTPS + Webhook | Payment notifications |

> **Note:** There is no integration between WordPress and Laravel. Each system operates independently. The REST API at `edufin.co.ke/api/v1` is served by Laravel via Nginx path routing on the main domain.

## Key Principles

1. **Separation of Concerns** - Content separate from transactions
2. **Security by Design** - Financial data isolated in Laravel
3. **API-First** - All logic exposed via versioned APIs
4. **Scalability** - Stateless design, horizontal scaling

---

**See Also:**
- [WordPress Architecture](./wordpress/README.md)
- [Laravel Architecture](./laravel/README.md)
