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
│              ┌────────────────────┼────────────────────┐                       │
│              │                    │                    │                        │
│              ▼                    ▼                    ▼                        │
│     ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│     │ edufin.co.ke    │  │app.edufin.co.ke │  │api.edufin.co.ke │             │
│     │   WORDPRESS     │  │    LARAVEL      │  │    LARAVEL      │             │
│     │  Landing Page   │  │  Client Portal  │  │    REST API     │             │
│     └────────┬────────┘  └────────┬────────┘  └────────┬────────┘             │
│              │                    │                    │                        │
│              │                    └──────────┬─────────┘                        │
│              │                               │                                  │
│              ▼                               ▼                                  │
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
- **Components:**
  - Client Portal (Livewire)
  - Admin Panel (Filament)
  - REST API (Sanctum)
  - Queue Workers (Horizon)
- **Contains:** Users, loans, KYC, documents, transactions

## Integration Points

| Integration | Protocol | Purpose |
|-------------|----------|---------|
| WordPress → Laravel | HTTPS + API Key | SSO, public data |
| Laravel → CBS | HTTPS + mTLS | Financial operations |
| Mobile → Laravel | HTTPS + JWT | Client access |
| CBS → Laravel | HTTPS + Webhook | Payment notifications |

## Key Principles

1. **Separation of Concerns** - Content separate from transactions
2. **Security by Design** - Financial data isolated in Laravel
3. **API-First** - All logic exposed via versioned APIs
4. **Scalability** - Stateless design, horizontal scaling

---

**See Also:**
- [WordPress Architecture](./wordpress/README.md)
- [Laravel Architecture](./laravel/README.md)
