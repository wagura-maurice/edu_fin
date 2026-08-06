# EduFin Platform - Architecture Advisory

## Technical Description: WordPress and Laravel as Independent Systems

**Date:** August 6, 2026  
**Version:** 1.0  
**Audience:** Technical Leadership, Development Team

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [WordPress System (edufin.co.ke)](#3-wordpress-system-edufincoke)
4. [Laravel System (app.edufin.co.ke)](#4-laravel-system-appedufincoke)
5. [Independence Principles](#5-independence-principles)
6. [Security Implications](#6-security-implications)
7. [Deployment & Operations](#7-deployment--operations)
8. [Implementation Roadmap](#8-implementation-roadmap)

---

## 1. Executive Summary

### Architecture

EduFin is delivered through **two entirely separate and independent systems** that share no code, no data, no services, and no infrastructure dependencies. Each system is hosted on its own domain, maintained by its own team, deployed through its own pipeline, and secured under its own posture.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       EDUFIN - TWO INDEPENDENT SYSTEMS                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                              CLOUDFLARE                                     │
│                    (DNS, CDN, WAF, SSL Termination)                         │
│                                   │                                         │
│              ┌────────────────────┴────────────────────┐                    │
│              │                                          │                    │
│              ▼                                          ▼                    │
│   ┌─────────────────────────┐              ┌─────────────────────────┐    │
│   │   edufin.co.ke          │              │   app.edufin.co.ke      │    │
│   │   WORDPRESS             │              │   LARAVEL               │    │
│   │   (Public Website)      │              │   (Application)         │    │
│   ├─────────────────────────┤              ├─────────────────────────┤    │
│   │ • Marketing Pages       │              │ • Login / Register      │    │
│   │ • Blog/News/Articles    │              │ • Client Portal         │    │
│   │ • SEO Optimization      │              │ • Loan Applications     │    │
│   │ • Product Listings      │              │ • KYC Workflows         │    │
│   │ • Landing Pages         │              │ • Statements            │    │
│   │ • /api/v1 → Laravel API │              │ • Admin Panel (Filament)│    │
│   └──────────┬──────────────┘              └──────────┬──────────────┘    │
│              │                                         │                    │
│              ▼                                         ▼                    │
│   ┌─────────────────────────┐              ┌─────────────────────────┐    │
│   │   WordPress Database    │              │   Laravel Database      │    │
│   │   (Content Only)        │              │   (Business Data)       │    │
│   └─────────────────────────┘              └──────────┬──────────────┘    │
│                                                       │                    │
│                                                       ▼                    │
│                                          ┌─────────────────────────┐      │
│                                          │   Core Banking API      │      │
│                                          │   (External)            │      │
│                                          └─────────────────────────┘      │
│                                                                             │
│   NO SHARED SERVICES • NO SHARED DATABASE • NO SSO • NO DATA SYNC           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Verdict

Treating WordPress and Laravel as two fully independent systems provides clean separation of concerns, clear security boundaries, and independent lifecycles. WordPress handles public marketing presence; Laravel handles all business operations. Neither system depends on the other to function.

---

## 2. Architecture Overview

### 2.1 Component Responsibilities

| System | Domain | Technology | Responsibilities |
|--------|--------|------------|------------------|
| Public Website | edufin.co.ke | WordPress | Marketing, SEO, Blog, CMS, Product pages |
| REST API | edufin.co.ke/api/v1 | Laravel (Sanctum) | Mobile API, webhooks, integrations (Nginx path routing on main domain) |
| Application | app.edufin.co.ke | Laravel (Blade/Livewire) | Login, Register, Dashboard, KYC, Loans, Statements, Admin (Filament) |
| Core Banking | (External) | Third-party | Financial transactions, disbursements |

### 2.2 Strengths of the Two-System Approach

| Aspect | Benefit |
|--------|---------|
| **SEO Maturity** | WordPress has 20+ years of SEO optimization; plugins like Yoast/RankMath |
| **Content Management** | Non-technical staff can manage content without developer involvement |
| **Time-to-Market** | WordPress themes accelerate public website delivery |
| **Plugin Ecosystem** | Forms, analytics, A/B testing, chat widgets readily available |
| **Laravel Excellence** | Best-in-class for complex business logic, API development, security |
| **Independent Lifecycles** | Each system can be released, scaled, and maintained on its own cadence |
| **Clear Ownership** | Marketing team owns WordPress; engineering team owns Laravel |

### 2.3 Considerations

| Aspect | Consideration |
|--------|---------------|
| **Two Codebases** | Different frameworks, patterns, deployment pipelines — by design |
| **Two Teams** | WordPress and Laravel require distinct skill sets |
| **Two Environments** | Separate hosting, optimization strategies, and server configurations |
| **No Cross-System Features** | Features that would require coupling are implemented within a single system |

---

## 3. WordPress System (edufin.co.ke)

### 3.1 Scope

The WordPress system is the public-facing brand presence for EduFin. It is a standalone website with no integration into the Laravel application.

- **Purpose:** Public-facing marketing and content
- **Managed by:** Secretarial/marketing staff
- **Contains:** Marketing content, blog, FAQs, product pages
- **Does NOT contain:** Business data, PII, financial information, user accounts for the application

### 3.2 WordPress Security Hardening

```php
// wp-config.php - Security hardening

// Disable file editing in admin
define('DISALLOW_FILE_EDIT', true);

// Disable plugin/theme installation (deploy via CI/CD only)
define('DISALLOW_FILE_MODS', true);

// Force SSL for admin
define('FORCE_SSL_ADMIN', true);

// Limit post revisions
define('WP_POST_REVISIONS', 5);

// Disable XML-RPC (common attack vector)
add_filter('xmlrpc_enabled', '__return_false');

// Custom auth keys (regenerate regularly)
define('AUTH_KEY',         'unique-phrase-here');
define('SECURE_AUTH_KEY',  'unique-phrase-here');
define('LOGGED_IN_KEY',    'unique-phrase-here');
define('NONCE_KEY',        'unique-phrase-here');
define('AUTH_SALT',        'unique-phrase-here');
define('SECURE_AUTH_SALT', 'unique-phrase-here');
define('LOGGED_IN_SALT',   'unique-phrase-here');
define('NONCE_SALT',       'unique-phrase-here');

// Move wp-content directory
define('WP_CONTENT_DIR', '/var/www/edufin-content');
define('WP_CONTENT_URL', 'https://cdn.edufin.co.ke/content');

// Custom database prefix (not wp_)
$table_prefix = 'edf_';
```

```nginx
# nginx configuration for WordPress security

server {
    # ... other config ...

    # Block access to sensitive files
    location ~* ^/(wp-config\.php|readme\.html|license\.txt) {
        deny all;
    }

    # Block PHP execution in uploads
    location ~* /wp-content/uploads/.*\.php$ {
        deny all;
    }

    # Block access to wp-includes
    location ~* ^/wp-includes/.*\.php$ {
        deny all;
    }

    # Disable XML-RPC
    location = /xmlrpc.php {
        deny all;
    }

    # Restrict wp-admin access by IP (if possible)
    location /wp-admin {
        allow 10.0.0.0/8;  # Internal network
        deny all;
    }

    # Rate limiting for wp-login
    location = /wp-login.php {
        limit_req zone=login burst=5 nodelay;
        include fastcgi_params;
        fastcgi_pass php-fpm;
    }
}
```

---

## 4. Laravel System (app.edufin.co.ke)

### 4.1 Scope

The Laravel system is the core business application. It is a standalone system with its own database, authentication, and integrations. It does not depend on WordPress.

- **Purpose:** All business operations
- **Managed by:** Engineering team
- **Domain:** `app.edufin.co.ke` (web portal + admin) and `edufin.co.ke/api/v1` (REST API via Nginx path routing on the main domain)
- **Components:**
  - Client Portal (Livewire) — `app.edufin.co.ke/dashboard`
  - Admin Panel (Filament) — `app.edufin.co.ke/admin`
  - REST API (Sanctum) for mobile clients — `edufin.co.ke/api/v1`
  - Queue Workers (Horizon)
- **Authentication:** All users (clients and staff) log in at `app.edufin.co.ke/login`. Onboarding/registration occurs at `app.edufin.co.ke/register`. After a successful login, the system redirects users to their respective dashboards based on their assigned roles:
  - Client → `app.edufin.co.ke/dashboard`
  - Staff (Loan Officer, KYC Verifier, System Admin, Super Admin) → `app.edufin.co.ke/admin`
- **Contains:** Users, loans, KYC, documents, transactions
- **Integrates with:** Core Banking System (external)

> **Note:** The REST API is served via Nginx path routing on the main domain (`edufin.co.ke/api/v1/...`) rather than a dedicated API subdomain. The admin panel (Filament) is served at the path `app.edufin.co.ke/admin` rather than a dedicated admin subdomain. All users share a single login interface at `app.edufin.co.ke/login` with role-based redirection.

### 4.2 Recommended Technology Stack

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LARAVEL TECHNOLOGY STACK                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  BACKEND                           FRONTEND                                 │
│  ───────                           ────────                                 │
│  • Laravel 11                      • Blade Templates                        │
│  • PHP 8.3                         • Livewire 3 (Portal)                   │
│  • PostgreSQL 16                   • Alpine.js                             │
│  • Redis (Cache/Queue)             • Tailwind CSS                          │
│                                                                             │
│  ADMIN & CMS                       INFRASTRUCTURE                           │
│  ──────────                        ──────────────                           │
│  • Filament 3                      • Ubuntu 22.04 LTS                      │
│  • Spatie Media Library            • Nginx                                 │
│                                    • PHP-FPM                               │
│  SEO & MARKETING                   • Cloudflare (CDN + WAF)                │
│  ───────────────                   • AWS S3 / Cloudflare R2               │
│  • spatie/laravel-sitemap                                                   │
│  • artesaos/seotools               MOBILE API                              │
│  • spatie/schema-org               ──────────                              │
│  • Google Tag Manager              • Laravel Sanctum                       │
│                                    • API Versioning                        │
│                                    • Rate Limiting                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 4.3 Integration Points

| Integration | Protocol | Purpose |
|-------------|----------|---------|
| Laravel → CBS | HTTPS + mTLS | Financial operations |
| Mobile → Laravel API (`edufin.co.ke/api/v1`) | HTTPS + JWT | Client access |
| CBS → Laravel API (`edufin.co.ke/api/v1`) | HTTPS + Webhook | Payment notifications |

Note: There is no integration between WordPress and Laravel. Each system operates independently. The REST API at `edufin.co.ke/api/v1` is served by Laravel via Nginx path routing on the main domain.

---

## 5. Independence Principles

The two systems are designed to be fully independent. The following principles are enforced:

1. **No Shared Database** — WordPress uses its own MySQL database for content; Laravel uses its own PostgreSQL database for business data. The two databases are hosted separately and neither system has credentials for the other.

2. **No Shared Authentication / SSO** — Each system maintains its own user accounts and authentication. WordPress admin accounts are for content editors; Laravel accounts are for clients and staff. There is no single sign-on between them.

3. **No Data Synchronization** — User data, content, and business data are not replicated between systems. WordPress does not consume Laravel APIs, and Laravel does not consume WordPress APIs.

4. **No Shared Services** — There are no shared queues, caches, message brokers, or service buses between the two systems. Each system has its own Redis instance (where applicable) and its own infrastructure.

5. **No Cross-System Functionality** — Features that would require the two systems to communicate are not implemented. Public-facing calls-to-action link to `app.edufin.co.ke/login` and `app.edufin.co.ke/register` via standard HTTP links; no authenticated handoff or token sharing occurs. The REST API at `edufin.co.ke/api/v1` is served by Laravel via Nginx path routing on the main domain, not via WordPress.

6. **Independent Deployment** — Each system has its own CI/CD pipeline, release cadence, and rollback procedure. A deployment to one system never affects the other.

7. **Independent Scaling** — Each system scales on its own. A traffic spike on the marketing site does not consume application resources, and vice versa.

---

## 6. Security Implications

### 6.1 WordPress Security Concerns for Fintech

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WORDPRESS SECURITY RISK ASSESSMENT                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  THREAT LANDSCAPE                                                           │
│  ────────────────                                                           │
│  • WordPress powers 43% of all websites → Primary target for attackers      │
│  • 90% of hacked CMS sites in 2023 were WordPress (Sucuri Report)          │
│  • Average WordPress site attacked 94 times/day (Wordfence)                │
│  • 52% of vulnerabilities come from plugins                                 │
│                                                                             │
│  RISK MATRIX FOR THE WORDPRESS SYSTEM                                       │
│  ──────────────────────────────────────────                                 │
│                                                                             │
│  ┌─────────────────┬────────────┬────────────┬─────────────────────────┐   │
│  │ Vulnerability   │ Likelihood │ Impact     │ Consequence             │   │
│  ├─────────────────┼────────────┼────────────┼─────────────────────────┤   │
│  │ Plugin RCE      │ High       │ High       │ Marketing site defaced  │   │
│  │ SQL Injection   │ Medium     │ High       │ Content data leak       │   │
│  │ XSS Attacks     │ High       │ Medium     │ Visitor session risk    │   │
│  │ Brute Force     │ High       │ Medium     │ Admin compromise        │   │
│  │ File Upload     │ Medium     │ High       │ Malware distribution    │   │
│  │ CSRF            │ Medium     │ Medium     │ Unauthorized actions    │   │
│  │ XML-RPC Abuse   │ High       │ Medium     │ DDoS amplification      │   │
│  └─────────────────┴────────────┴────────────┴─────────────────────────┘   │
│                                                                             │
│  KEY MITIGATION: WordPress holds NO financial or PII data, so a            │
│  compromise is contained to public marketing content.                      │
│                                                                             │
│  REGULATORY IMPLICATIONS                                                    │
│  ───────────────────────                                                    │
│  • Data Protection Act 2019: Breach notification within 72 hours           │
│  • CBK Guidelines: Security controls for financial data (Laravel only)     │
│  • PCI-DSS: No payment data touches WordPress                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Network Isolation Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NETWORK ARCHITECTURE - INDEPENDENT SYSTEMS                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                              INTERNET                                        │
│                                 │                                           │
│                                 ▼                                           │
│                        ┌───────────────┐                                    │
│                        │   Cloudflare  │                                    │
│                        │   WAF + CDN   │                                    │
│                        └───────┬───────┘                                    │
│                                │                                            │
│         ┌──────────────────────┴──────────────────────┐                    │
│         │                                            │                      │
│         ▼                                            ▼                      │
│  ┌─────────────┐                            ┌─────────────────────┐        │
│  │  WordPress  │                            │   LARAVEL CLUSTER   │        │
│  │ edufin.co.ke│                            │ app.edufin.co.ke    │        │
│  ├─────────────┤                            ├─────────────────────┤        │
│  │  (Isolated) │                            │  ┌─────────┐       │        │
│  │             │                            │  │ Portal  │       │        │
│  │ • No access │                            │  │ Workers │       │        │
│  │   to Laravel│                            │  └────┬────┘       │        │
│  │   database  │                            │       │            │        │
│  │             │     /api/v1 (Nginx proxy)  │  ┌────┴────┐        │        │
│  │ • /api/v1 ──┼───────────────────────────►│  │   API   │       │        │
│  │   to Laravel│                            │  │ Workers │       │        │
│  └──────┬──────┘                            │  └────┬────┘       │        │
│         │                                   │       │            │        │
│         ▼                                   │       ▼            │        │
│  ┌─────────────┐                            │  ┌─────────────┐   │        │
│  │  WordPress  │                            │  │  Laravel DB │   │        │
│  │  Database   │                            │  │ (Financial) │   │        │
│  │ (Content)   │                            │  └─────────────┘   │        │
│  └─────────────┘                            └─────────┬──────────┘        │
│                                                       │                    │
│                                                       ▼                    │
│                                            ┌─────────────────────┐        │
│                                            │  Core Banking API   │        │
│                                            │  (External System)  │        │
│                                            └─────────────────────┘        │
│                                                                             │
│  KEY SECURITY BOUNDARIES:                                                   │
│  • WordPress CANNOT access Laravel database                                │
│  • WordPress CANNOT call Core Banking API                                  │
│  • WordPress PHP has no direct network path to Laravel infrastructure      │
│  • /api/v1 on edufin.co.ke is reverse-proxied by Nginx to Laravel (no      │
│    WordPress code involvement)                                             │
│  • Financial data never touches WordPress                                  │
│  • Each system is compromised independently with no lateral movement       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.3 Security Comparison

| Security Aspect | WordPress System | Laravel System |
|-----------------|------------------|----------------|
| Attack Surface | Large (public CMS) | Minimal (authenticated app) |
| Plugin Vulnerabilities | High Risk | N/A |
| Data Sensitivity | Low (marketing content) | High (financial/PII) |
| Security Updates | Frequent/Urgent | Predictable |
| Audit Complexity | Simple (content only) | Standard (financial) |
| Compliance Effort | Minimal | CBK / Data Protection Act |
| Incident Response | Single system | Single system |
| Blast Radius | Marketing content only | Business operations |

Because the two systems are fully isolated, a compromise of the WordPress system cannot reach Laravel infrastructure or data, and vice versa.

---

## 7. Deployment & Operations

### 7.1 Delivery Characteristics

| Factor | WordPress System | Laravel System |
|--------|------------------|----------------|
| Environments | 1 (per stage) | 1 (per stage) |
| CI/CD Pipelines | 1 | 1 |
| Database | 1 (MySQL) | 1 (PostgreSQL) |
| SSL/Domain | edufin.co.ke (+ /api/v1 path routing) | app.edufin.co.ke |
| Deployment Owner | Marketing/Web team | Engineering team |
| Testing Strategy | Theme/plugin QA | Automated test suite |

### 7.2 Maintenance Overhead

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ANNUAL MAINTENANCE EFFORT                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  WORDPRESS SYSTEM (edufin.co.ke)                                            │
│  ├── WordPress Updates  ████████████░░░░░░░░ 60 hrs/year                   │
│  ├── WP Security        ████████████████░░░░ 80 hrs/year                   │
│  ├── Plugin Updates     ████████████░░░░░░░░ 60 hrs/year                   │
│  └── Infrastructure     ████░░░░░░░░░░░░░░░░ 20 hrs/year                   │
│      TOTAL: ~220 hrs/year                                                   │
│                                                                             │
│  LARAVEL SYSTEM (app.edufin.co.ke)                                          │
│  ├── Framework Updates  ████████░░░░░░░░░░░░ 40 hrs/year                   │
│  ├── Security Patches   ████░░░░░░░░░░░░░░░░ 20 hrs/year                   │
│  ├── Package Updates    ██████░░░░░░░░░░░░░░ 30 hrs/year                   │
│  └── Infrastructure     ████░░░░░░░░░░░░░░░░ 20 hrs/year                   │
│      TOTAL: ~110 hrs/year                                                   │
│                                                                             │
│  NOTE: No cross-system maintenance overhead, because the two systems       │
│  do not integrate. There is no sync logic, no shared auth, and no          │
│  coupled release coordination to maintain.                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.3 Scalability Considerations

| Scenario | WordPress System | Laravel System |
|----------|------------------|----------------|
| Traffic Spike (Marketing) | Scale WordPress independently | Unaffected |
| Traffic Spike (Portal) | Unaffected | Scale Laravel independently |
| Database Scaling | Independent MySQL scaling | Independent PostgreSQL scaling |
| Caching Strategy | WP Object Cache | Laravel Cache (Redis) |
| CDN Integration | Cloudflare for edufin.co.ke | Cloudflare for app.edufin.co.ke |

---

## 8. Implementation Roadmap

### 8.1 WordPress System Setup

```
Phase 1 (Weeks 1-3): WordPress Foundation
├── Provision hosting for edufin.co.ke
├── Install WordPress with security hardening (see Section 3.2)
├── Configure Cloudflare DNS, CDN, and WAF for edufin.co.ke
├── Deploy theme and core plugins via CI/CD
└── Set up MySQL database (content only)

Phase 2 (Weeks 4-6): Content & SEO
├── Build marketing pages, product listings, landing pages
├── Configure SEO (Yoast/RankMath, sitemaps, structured data)
├── Set up blog and content workflows for editorial staff
├── Add calls-to-action linking to app.edufin.co.ke/login and app.edufin.co.ke/register
└── Performance and security review
```

### 8.2 Laravel System Setup

```
Phase 1 (Weeks 1-4): Laravel Foundation
├── Provision hosting for app.edufin.co.ke
├── Set up Laravel project with Filament
├── Implement authentication (Sanctum/Fortify) with role-based redirect
│   ├── Login at app.edufin.co.ke/login (all users)
│   ├── Register at app.edufin.co.ke/register (clients)
│   ├── Client → app.edufin.co.ke/dashboard
│   └── Staff → app.edufin.co.ke/admin (Filament)
├── Configure Nginx path routing: edufin.co.ke/api/v1 → Laravel (reverse proxy)
├── Configure PostgreSQL database (business data)
└── Configure Cloudflare DNS, CDN, and WAF for app.edufin.co.ke

Phase 2 (Weeks 5-8): Client Portal
├── Build dashboard with Livewire
├── Implement KYC workflows
├── Create loan application flows
├── Build statement generation
└── Integrate with Core Banking API

Phase 3 (Weeks 9-12): Admin & Mobile API
├── Build admin panel with Filament at app.edufin.co.ke/admin
├── Build RESTful API for mobile at edufin.co.ke/api/v1
├── Implement API authentication (Sanctum)
├── Create API documentation (Scribe)
└── Performance testing and security audit

Phase 4 (Weeks 13-16): Launch
├── Production deployment
├── Monitoring and alerting
├── Security audit sign-off
└── Go-live
```

### 8.3 SEO Checklist for the WordPress System

- [ ] Dynamic meta tags (title, description, keywords)
- [ ] Open Graph and Twitter Card tags
- [ ] Canonical URLs
- [ ] XML Sitemap (auto-generated)
- [ ] robots.txt configuration
- [ ] Structured data (JSON-LD)
- [ ] Breadcrumb navigation
- [ ] Clean URL structure
- [ ] Image optimization (WebP, lazy loading)
- [ ] Page speed optimization (< 3s LCP)
- [ ] Mobile responsiveness
- [ ] Internal linking strategy
- [ ] Google Search Console integration
- [ ] Analytics integration

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Technical Team | Initial advisory document |
