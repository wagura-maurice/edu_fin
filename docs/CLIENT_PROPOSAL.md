# EduFin Platform - Technical Requirements & Project Proposal

## Infrastructure, Hosting Strategy & Project Pricing

**Document Type:** Client Proposal  
**Date:** August 6, 2026  
**Version:** 1.0  
**Prepared For:** EduFin Management  
**Classification:** Business Confidential

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Domain & DNS Management](#2-domain--dns-management)
3. [Development Environment](#3-development-environment)
4. [Production Environment](#4-production-environment)
5. [Technical Implementation Strategy](#5-technical-implementation-strategy)
6. [Project Costing & Deliverables](#6-project-costing--deliverables)
7. [Timeline & Milestones](#7-timeline--milestones)
8. [Terms & Conditions](#8-terms--conditions)
9. [Next Steps](#9-next-steps)

---

## 1. Executive Summary

This document outlines the technical infrastructure requirements, hosting strategy, and project pricing for the EduFin education financing platform. The proposed solution delivers a robust, scalable, and cost-effective architecture comprising:

- **WordPress Website** - Company landing page and marketing content
- **Laravel Application** - Client portal, administration panel, and REST API

Our approach prioritizes:

| Priority | Implementation |
|----------|----------------|
| **Cost Efficiency** | Optimized infrastructure with minimal overhead |
| **Scalability** | Clear upgrade path from development to production |
| **Security** | Enterprise-grade protection via Cloudflare |
| **Future-Readiness** | API architecture prepared for mobile app integration |

---

## 2. Domain & DNS Management

### 2.1 Proposed Solution: Cloudflare

We recommend **Cloudflare** as the centralized DNS and security management platform for all EduFin domains and subdomains.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           CLOUDFLARE DNS ARCHITECTURE                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                              CLOUDFLARE ACCOUNT                                 │
│                           (Client-Owned & Managed)                              │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                         │  │
│  │  DOMAIN: edufin.co.ke                                                  │  │
│  │                                                                         │  │
│  │  DNS Records:                                                          │  │
│  │  ─────────────                                                         │  │
│  │  edufin.co.ke        A       → Server IP    (WordPress + API routing)│  │
│  │  www.edufin.co.ke    CNAME   → edufin.co.ke (Redirect)                │  │
│  │  app.edufin.co.ke    A       → Server IP    (Laravel Portal + Admin)  │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  INCLUDED SERVICES (Free Tier):                                                │
│  • DNS Management                • SSL/TLS Certificates (Auto)                │
│  • DDoS Protection (Basic)       • CDN for Static Assets                      │
│  • Web Application Firewall      • Analytics & Insights                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Benefits

| Benefit | Description |
|---------|-------------|
| **High Availability** | Global anycast network with 99.99% uptime SLA |
| **Security** | DDoS protection, WAF, and bot management |
| **Performance** | CDN caching reduces server load and improves page speed |
| **SSL/TLS** | Free automatic HTTPS certificates |
| **Centralized Management** | Single dashboard for all domains and subdomains |

### 2.3 Client Responsibility

> **Important:** The client is responsible for providing and maintaining the Cloudflare account to ensure full ownership and control of the domain infrastructure.

**Required from Client:**
- Cloudflare account credentials (or invitation to manage)
- Domain registrar access to update nameservers
- Approval for DNS configuration changes

**We Will Provide:**
- Complete DNS configuration
- SSL/TLS setup and optimization
- Security rule configuration
- Ongoing technical support

---

## 3. Development Environment

### 3.1 Proposed Infrastructure

For the development and testing phase, we propose a **Contabo VPS** (Virtual Private Server) that provides excellent value while meeting all technical requirements.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        DEVELOPMENT SERVER SPECIFICATION                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PROVIDER:        Contabo VPS                                                  │
│  PLAN:            Cloud VPS S                                                  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                         │  │
│  │  SPECIFICATIONS                                                        │  │
│  │  ──────────────                                                        │  │
│  │                                                                         │  │
│  │  vCPUs:           4 Cores                                              │  │
│  │  RAM:             8 GB                                                 │  │
│  │  Storage:         100 GB SSD                                           │  │
│  │  Bandwidth:       Unlimited (Fair Use)                                 │  │
│  │  Operating System: Ubuntu 22.04 LTS                                    │  │
│  │                                                                         │  │
│  │  MONTHLY COST:    ~$6 USD (~KSH 780)                                   │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  HOSTED APPLICATIONS:                                                          │
│  • WordPress (Landing Page)      → dev.edufin.co.ke                           │
│  • Laravel Application           → dev-app.edufin.co.ke                       │
│  • MySQL Database (WordPress)                                                  │
│  • PostgreSQL Database (Laravel)                                              │
│  • Redis Cache                                                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Development Environment Purpose

| Purpose | Description |
|---------|-------------|
| **Development** | Active development and coding |
| **Testing** | Quality assurance and bug fixing |
| **Client Review** | Staging environment for client approval |
| **Training** | Staff training before production launch |

### 3.3 DNS Configuration (Development)

Cloudflare will be configured to point development subdomains to this server:

| Subdomain | Purpose |
|-----------|---------|
| `dev.edufin.co.ke` | WordPress development site |
| `dev-app.edufin.co.ke` | Laravel portal development |

---

## 4. Production Environment

### 4.1 Proposed Infrastructure

For production deployment, we recommend upgrading to a **Contabo VDS** (Virtual Dedicated Server) to ensure enhanced performance, dedicated resources, and production-grade reliability.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        PRODUCTION SERVER SPECIFICATION                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PROVIDER:        Contabo VDS                                                  │
│  PLAN:            Virtual Dedicated Server L                                   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                         │  │
│  │  SPECIFICATIONS                                                        │  │
│  │  ──────────────                                                        │  │
│  │                                                                         │  │
│  │  vCPUs:           6 Dedicated Cores                                    │  │
│  │  RAM:             24 GB                                                │  │
│  │  Storage:         180 GB NVMe SSD                                      │  │
│  │  Bandwidth:       Unlimited (Fair Use)                                 │  │
│  │  Operating System: Ubuntu 22.04 LTS                                    │  │
│  │                                                                         │  │
│  │  MONTHLY COST:    ~$40 USD (~KSH 5,200)                                │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  KEY ADVANTAGES OVER VPS:                                                      │
│  • Dedicated CPU resources (not shared)                                       │
│  • Higher RAM for concurrent users                                            │
│  • NVMe storage for faster database operations                                │
│  • Better suited for financial application workloads                          │
│                                                                                 │
│  HOSTED APPLICATIONS:                                                          │
│  • WordPress (Landing Page)      → edufin.co.ke                               │
│  • Laravel Application           → app.edufin.co.ke                           │
│  • Laravel API                   → edufin.co.ke/api/v1 (path routing)         │
│  • MySQL Database (WordPress)                                                  │
│  • PostgreSQL Database (Laravel)                                              │
│  • Redis Cache & Sessions                                                      │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 Production DNS Configuration

Cloudflare will serve as the primary DNS and security layer:

| Domain | Application | Purpose |
|--------|-------------|---------|
| `edufin.co.ke` | WordPress + API | Public marketing website + API path routing |
| `www.edufin.co.ke` | Redirect | Redirects to root domain |
| `app.edufin.co.ke` | Laravel | Client portal + admin panel (Filament) |

### 4.3 Development vs. Production Comparison

| Aspect | Development (VPS) | Production (VDS) |
|--------|-------------------|------------------|
| **CPU** | 4 vCPUs (Shared) | 6 vCPUs (Dedicated) |
| **RAM** | 8 GB | 24 GB |
| **Storage** | 100 GB SSD | 180 GB NVMe |
| **Monthly Cost** | ~$6 (~KSH 780) | ~$40 (~KSH 5,200) |
| **Use Case** | Development, Testing | Live Production |
| **Concurrent Users** | ~50 | ~500+ |

---

## 5. Technical Implementation Strategy

### 5.1 Co-Hosting Architecture

Both the WordPress website and Laravel application will be hosted on a **single server instance** for both development and production environments. This approach is achieved through careful server configuration and resource isolation.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        SINGLE SERVER CO-HOSTING STRATEGY                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                              CONTABO SERVER                                     │
│                         (VPS for Dev / VDS for Prod)                           │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           NGINX (Reverse Proxy)                         │  │
│  │                                                                         │  │
│  │  Routes requests based on domain:                                      │  │
│  │  • edufin.co.ke        → WordPress (PHP-FPM 8.2)                       │  │
│  │    • /api/             → Laravel (PHP-FPM 8.3) [path routing]          │  │
│  │  • app.edufin.co.ke    → Laravel (PHP-FPM 8.3)                         │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                            │
│              ┌────────────────────┼────────────────────┐                      │
│              │                    │                    │                       │
│              ▼                    ▼                    ▼                       │
│  ┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐         │
│  │                   │  │                   │  │                   │         │
│  │    WORDPRESS      │  │     LARAVEL       │  │      REDIS        │         │
│  │                   │  │                   │  │                   │         │
│  │  • PHP-FPM 8.2    │  │  • PHP-FPM 8.3    │  │  • Sessions       │         │
│  │  • /var/www/wp    │  │  • /var/www/app   │  │  • Cache          │         │
│  │                   │  │  • Horizon Queue  │  │  • Queues         │         │
│  │                   │  │  • Scheduler      │  │                   │         │
│  │                   │  │                   │  │                   │         │
│  └─────────┬─────────┘  └─────────┬─────────┘  └───────────────────┘         │
│            │                      │                                           │
│            ▼                      ▼                                           │
│  ┌───────────────────┐  ┌───────────────────┐                                │
│  │      MySQL        │  │    PostgreSQL     │                                │
│  │   (WordPress)     │  │     (Laravel)     │                                │
│  │                   │  │                   │                                │
│  │  Content only     │  │  All business     │                                │
│  │  No PII           │  │  data & PII       │                                │
│  └───────────────────┘  └───────────────────┘                                │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Key Technical Considerations

| Consideration | Implementation |
|---------------|----------------|
| **Process Isolation** | Separate PHP-FPM pools for WordPress and Laravel |
| **Database Separation** | MySQL for WordPress, PostgreSQL for Laravel |
| **Resource Allocation** | PHP memory limits configured per application |
| **Security Isolation** | Separate Linux users and file permissions |
| **SSL/TLS** | Cloudflare handles all SSL termination |
| **Caching** | Redis shared but with separate databases |

### 5.3 Benefits of Co-Hosting

| Benefit | Description |
|---------|-------------|
| **Cost Efficiency** | Single server reduces infrastructure costs by ~60% |
| **Simplified Management** | One server to maintain, monitor, and backup |
| **Reduced Latency** | Internal communication between services is faster |
| **Easier Deployment** | Streamlined CI/CD pipeline |

### 5.4 Scalability Path

Should traffic demands increase significantly, the architecture supports easy migration to separate servers:

```
CURRENT (Single Server)          FUTURE (Separate Servers)
─────────────────────────        ─────────────────────────
┌─────────────────────┐          ┌─────────────────────┐
│  WordPress + Laravel │    →    │     WordPress       │
│  (Single VDS)        │          │     (VPS)           │
└─────────────────────┘          └─────────────────────┘
                                  ┌─────────────────────┐
                                  │      Laravel        │
                                  │      (VDS)          │
                                  └─────────────────────┘
```

---

## 6. Project Costing & Deliverables

### 6.1 Development Costs

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           PROJECT COSTING SUMMARY                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                         │  │
│  │  1. WORDPRESS WEBSITE                                    KSH 50,000    │  │
│  │     ─────────────────────────────────────────────────────────────────  │  │
│  │                                                                         │  │
│  │     Deliverables:                                                      │  │
│  │     • Custom responsive theme (EduFin branding)                        │  │
│  │     • Homepage with hero section, features, CTAs                       │  │
│  │     • About Us page (company profile, team, mission)                   │  │
│  │     • Products/Services page (financing packages)                      │  │
│  │     • Blog/News section                                                │  │
│  │     • FAQ page with categorized questions                              │  │
│  │     • Contact page with form integration                               │  │
│  │     • Legal pages (Terms & Conditions, Privacy Policy)                 │  │
│  │     • SEO optimization (Yoast SEO configuration)                       │  │
│  │     • Performance optimization (caching, image optimization)           │  │
│  │     • Live chat widget integration                                     │  │
│  │     • Newsletter signup integration                                    │  │
│  │     • Mobile-responsive design                                         │  │
│  │     • Admin training for secretarial staff                             │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                         │  │
│  │  2. LARAVEL APPLICATION                                  KSH 150,000   │  │
│  │     ─────────────────────────────────────────────────────────────────  │  │
│  │                                                                         │  │
│  │     Deliverables:                                                      │  │
│  │                                                                         │  │
│  │     A. CLIENT PORTAL (Livewire)                                        │  │
│  │        • User registration and authentication                          │  │
│  │        • Email/Phone verification (OTP)                                │  │
│  │        • Profile management                                            │  │
│  │        • KYC document upload and tracking                              │  │
│  │        • Education beneficiary management                              │  │
│  │        • Loan application workflow                                     │  │
│  │        • Collateral registration                                       │  │
│  │        • Payment schedule viewing                                      │  │
│  │        • Statement generation and download                             │  │
│  │        • Notification center                                           │  │
│  │        • Responsive dashboard design                                   │  │
│  │                                                                         │  │
│  │     B. ADMINISTRATION PANEL (Filament)                                 │  │
│  │        • User and role management                                      │  │
│  │        • Loan application review queue                                 │  │
│  │        • KYC verification workflow                                     │  │
│  │        • Collateral approval process                                   │  │
│  │        • Client account management                                     │  │
│  │        • System configuration                                          │  │
│  │        • Reports and analytics dashboard                               │  │
│  │        • Audit log viewer                                              │  │
│  │                                                                         │  │
│  │     C. REST API (For Future Mobile App)                                │  │
│  │        • Authentication endpoints (JWT)                                │  │
│  │        • Profile management endpoints                                  │  │
│  │        • Loan application endpoints                                    │  │
│  │        • Beneficiary management endpoints                              │  │
│  │        • Document upload endpoints                                     │  │
│  │        • Notification endpoints                                        │  │
│  │        • API documentation (OpenAPI/Swagger)                           │  │
│  │        • Rate limiting and security                                    │  │
│  │                                                                         │  │
│  │     D. CORE BANKING INTEGRATION                                        │  │
│  │        • API integration architecture                                  │  │
│  │        • Loan account synchronization                                  │  │
│  │        • Payment webhook processing                                    │  │
│  │        • Reconciliation framework                                      │  │
│  │        • Error handling and retry logic                                │  │
│  │                                                                         │  │
│  │     E. INFRASTRUCTURE & SECURITY                                       │  │
│  │        • Server setup and configuration                                │  │
│  │        • Database design and optimization                              │  │
│  │        • Redis caching implementation                                  │  │
│  │        • Queue system (Horizon)                                        │  │
│  │        • SSL/TLS configuration                                         │  │
│  │        • Security hardening                                            │  │
│  │        • Backup configuration                                          │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                                                                         │  │
│  │  TOTAL PROJECT COST                                      KSH 200,000   │  │
│  │                                                                         │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Recurring Infrastructure Costs

| Phase | Server | Monthly Cost (USD) | Monthly Cost (KSH)* |
|-------|--------|-------------------|---------------------|
| Development | Contabo VPS | ~$6 | ~KSH 780 |
| Production | Contabo VDS | ~$40 | ~KSH 5,200 |

*Exchange rate: 1 USD = 130 KSH (approximate)

### 6.3 Exclusions

> **Note:** The following items are **NOT included** in this quotation and will be quoted separately:

| Item | Status |
|------|--------|
| **Mobile Application (Flutter)** | To be quoted at a later date |
| **Core Banking System** | Client to provide or procure separately |
| **Domain Registration** | Client responsibility |
| **Cloudflare Account** | Client responsibility |
| **Third-party API Costs** | SMS, Email services (pay-as-you-go) |
| **Content Creation** | Text, images, videos for website |

---

## 7. Timeline & Milestones

### 7.1 Proposed Timeline

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            PROJECT TIMELINE                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PHASE 1: FOUNDATION (Weeks 1-2)                                               │
│  ─────────────────────────────────                                             │
│  □ Development server setup                                                    │
│  □ Cloudflare DNS configuration                                                │
│  □ Database setup (MySQL + PostgreSQL)                                         │
│  □ Development environment ready                                               │
│                                                                                 │
│  PHASE 2: WORDPRESS DEVELOPMENT (Weeks 3-4)                                    │
│  ──────────────────────────────────────────                                    │
│  □ Custom theme development                                                    │
│  □ Page templates and content structure                                        │
│  □ Plugin configuration                                                        │
│  □ Client review and feedback                                                  │
│                                                                                 │
│  PHASE 3: LARAVEL CORE (Weeks 5-8)                                             │
│  ─────────────────────────────────                                             │
│  □ Database schema and migrations                                              │
│  □ Authentication system                                                       │
│  □ Client portal (Livewire)                                                    │
│  □ Admin panel (Filament)                                                      │
│  □ Core business logic                                                         │
│                                                                                 │
│  PHASE 4: API & INTEGRATION (Weeks 9-10)                                       │
│  ───────────────────────────────────────                                       │
│  □ REST API development                                                        │
│  □ Core banking integration framework                                          │
│  □ Webhook handlers                                                            │
│  □ API documentation                                                           │
│                                                                                 │
│  PHASE 5: TESTING & QA (Weeks 11-12)                                           │
│  ───────────────────────────────────                                           │
│  □ Comprehensive testing                                                       │
│  □ Security audit                                                              │
│  □ Performance optimization                                                    │
│  □ Bug fixes                                                                   │
│                                                                                 │
│  PHASE 6: PRODUCTION DEPLOYMENT (Week 13)                                      │
│  ────────────────────────────────────────                                      │
│  □ Production server setup (VDS)                                               │
│  □ Data migration                                                              │
│  □ DNS cutover                                                                 │
│  □ Go-live                                                                     │
│                                                                                 │
│  PHASE 7: HANDOVER & TRAINING (Week 14)                                        │
│  ──────────────────────────────────────                                        │
│  □ Staff training                                                              │
│  □ Documentation handover                                                      │
│  □ Support transition                                                          │
│                                                                                 │
│  TOTAL DURATION: 14 WEEKS (3.5 MONTHS)                                         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Payment Schedule

| Milestone | Percentage | Amount (KSH) | Due |
|-----------|------------|--------------|-----|
| Project Kickoff | 30% | 60,000 | Upon signing |
| WordPress Completion | 20% | 40,000 | Week 4 |
| Laravel Portal Completion | 25% | 50,000 | Week 8 |
| API & Integration Complete | 15% | 30,000 | Week 10 |
| Go-Live & Handover | 10% | 20,000 | Week 14 |
| **Total** | **100%** | **200,000** | |

---

## 8. Terms & Conditions

### 8.1 Scope

- This quotation covers the deliverables explicitly listed in Section 6
- Any additional features or changes will be quoted separately
- Mobile application development is excluded and will be quoted separately

### 8.2 Client Responsibilities

- Provide Cloudflare account access
- Provide domain registrar access for nameserver updates
- Provide content (text, images, logos) for the website
- Timely feedback and approvals at each milestone
- Provide Core Banking System API documentation and credentials

### 8.3 Support & Maintenance

- 30-day post-launch support included
- Bug fixes during support period at no additional cost
- Extended support and maintenance available (quoted separately)

### 8.4 Intellectual Property

- All custom code developed becomes client property upon final payment
- Third-party libraries remain under their respective licenses
- Documentation and training materials included

### 8.5 Validity

- This quotation is valid for **30 days** from the date of issue
- Prices are subject to change after validity period

---

## 9. Next Steps

To proceed with this project, we require:

| Step | Action Required | Responsible |
|------|-----------------|-------------|
| 1 | Review and approve this proposal | Client |
| 2 | Sign project agreement | Both parties |
| 3 | Provide Cloudflare account access | Client |
| 4 | Initial payment (30%) | Client |
| 5 | Project kickoff meeting | Both parties |

---

## Contact Information

For questions or clarifications regarding this proposal, please contact:

**[Developer/Company Name]**  
Email: [email]  
Phone: [phone]

---

**Document Prepared By:** [Name]  
**Date:** August 6, 2026  
**Version:** 1.0

---

*This document is confidential and intended solely for the use of EduFin Management. Unauthorized distribution is prohibited.*
