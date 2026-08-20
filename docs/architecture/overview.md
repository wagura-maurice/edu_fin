# EduFin - Architecture Overview

## System Architecture

**Version:** 2.0  
**Last Updated:** August 10, 2026

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
│     │   WORDPRESS     │  /api/v1 (proxy)   │    LARAVEL      │                 │
│     │  Landing Page + │───────────────────►│  Client Portal  │                 │
│     │  Onboarding UI  │  register + options│  Admin (Filament)│                 │
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

### WordPress (Landing Page + Onboarding Interface)
- **Purpose:** Public-facing brand presence **and client onboarding interface**
- **Managed by:** Secretarial staff
- **Contains:** Marketing content, blog, FAQs, support chat widget (bottom-right), **onboarding wizard** (`/get-started`)
- **Onboarding role:** Hosts the multi-step "Get Started" wizard that collects client registration data and submits it to the Laravel registration API. Fetches dynamic dropdown options (Location, Gender, Employment Type, etc.) from Laravel's `/options/*` endpoints.
- **Does NOT contain:** Business data, PII storage, financial information — the wizard collects and forwards data to Laravel but does not persist it

### Laravel (Core Application)
- **Purpose:** All business operations
- **Domain:** `app.edufin.co.ke` (portal + admin) and `edufin.co.ke/api/v1` (REST API via Nginx path routing on the main domain)
- **Components:**
  - Client Portal (Livewire) — `app.edufin.co.ke/dashboard`
  - Admin Panel (Filament) — `app.edufin.co.ke/admin`
  - REST API (Sanctum) — `edufin.co.ke/api/v1`
  - Queue Workers (Horizon)
- **Authentication:** Login at `app.edufin.co.ke/login`. **No front-end `/register` page** — registration is API-only (`POST edufin.co.ke/api/v1/auth/register`), consumed by the WordPress onboarding wizard and future mobile apps. Laravel UI is limited to `/login`, `/forgot-password`, and `/password/reset`. After login, users are redirected by role:
  - Client → `app.edufin.co.ke/dashboard`
  - Staff (Loan Officer, KYC Verifier, System Admin, Super Admin) → `app.edufin.co.ke/admin`
- **Registration role:** Owns the data logic and dynamic options for onboarding. Exposes `/options/*` endpoints (locations, genders, employment types, etc.) consumed by the WordPress wizard.
- **Contains:** Users, loans, KYC, documents, transactions

## Integration Points

| Integration | Protocol | Purpose |
|-------------|----------|---------|
| Laravel → CBS | HTTPS + mTLS | Financial operations |
| Mobile → Laravel API (`edufin.co.ke/api/v1`) | HTTPS + JWT | Client access |
| CBS → Laravel API (`edufin.co.ke/api/v1`) | HTTPS + Webhook | Payment notifications |
| AI Agents → WordPress | HTTPS REST + SSH + WebSocket | Content management, SEO suggestions, support chat widget, health monitoring |
| AI Agents → Laravel | HTTPS REST + SSH | Product data, audit logging, health monitoring |
| AI Agents → External | HTTPS + SMTP/IMAP + SSH | Social media (X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube), email, WhatsApp (WAHA), Git, server access |

> **Note:** WordPress and Laravel are connected by a REST API contract for onboarding. WordPress consumes Laravel's `POST /api/v1/auth/register` endpoint (for registration) and `GET /api/v1/options/*` endpoints (for dynamic dropdown data). The REST API at `edufin.co.ke/api/v1` is served by Laravel via Nginx path routing on the main domain. There is no SSO and no shared sessions — the registration API is consumer-agnostic (WordPress today, mobile apps in the future).
>
> **AI Agents Layer:** The AI Agents module is a non-invasive intelligence layer that interacts with both WordPress and Laravel through their existing APIs and SSH access. It does not modify core business logic or access PII/financial data. See [AI Agents Architecture](./ai-agents/README.md) for full details.

## Key Principles

1. **Separation of Concerns** - Content + onboarding UI (WordPress) separate from data logic (Laravel)
2. **Security by Design** - Financial data isolated in Laravel; WordPress collects but does not persist PII
3. **API-First** - All logic exposed via versioned APIs; registration is API-only (no Laravel /register page)
4. **Consumer-Agnostic Registration** - The registration API serves WordPress today and mobile apps tomorrow
5. **Scalability** - Stateless design, horizontal scaling
6. **Non-Invasive Agent Layer** - AI agents interact via existing APIs; no core modifications

---

**See Also:**
- [WordPress Architecture](./wordpress/README.md)
- [Laravel Architecture](./laravel/README.md)
- [AI Agents Architecture](./ai-agents/README.md)
