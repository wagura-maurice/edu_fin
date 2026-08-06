# EduFin - Site Links Reference

**Version:** 1.1  
**Last Updated:** August 6, 2026

---

## Overview

EduFin operates as two entirely separate and independent systems. This document lists the public URLs, page routes, and API endpoints for each system.

| System | Domain | Technology | Purpose |
|--------|--------|------------|---------|
| Public Website | `edufin.co.ke` | WordPress | Marketing, SEO, blog, content |
| Application | `app.edufin.co.ke` | Laravel (Livewire) | Client portal, admin panel, KYC, loans, statements |
| REST API | `edufin.co.ke/api/v1` | Laravel | Mobile app, webhooks, integrations (path-based on main domain) |
| Static Assets | `cdn.edufin.co.ke` | Cloudflare R2 | Media, images, CSS, JS |

> **Note:** The two systems are independent. WordPress links to the Laravel application only via standard public HTTP links (e.g., "Login" / "Get Started" buttons). There is no SSO, no shared sessions, and no API integration between them.
>
> **API Routing:** The REST API is served via a path-based prefix on the main domain (`edufin.co.ke/api/v1/...`) rather than a dedicated API subdomain. Nginx routes `/api/` requests on `edufin.co.ke` to the Laravel application.
>
> **Authentication:** All users (clients and staff) access the login interface at `app.edufin.co.ke/login`. Onboarding/registration occurs at `app.edufin.co.ke/register`. After a successful login, the system redirects users to their respective dashboards based on their assigned roles.

---

## 1. WordPress Site (edufin.co.ke)

### 1.1 Public Pages

| URL | Page | Description |
|-----|------|-------------|
| `https://edufin.co.ke/` | Home | Landing page |
| `https://edufin.co.ke/products` | Products | Education financing packages |
| `https://edufin.co.ke/how-it-works` | How It Works | Process explanation |
| `https://edufin.co.ke/calculator` | Calculator | Loan calculator widget |
| `https://edufin.co.ke/blog` | Blog | Articles, news, announcements |
| `https://edufin.co.ke/contact` | Contact | Contact form and details |
| `https://edufin.co.ke/about` | About Us | Company information |
| `https://edufin.co.ke/faq` | FAQs | Help content |
| `https://edufin.co.ke/testimonials` | Testimonials | Client reviews |
| `https://edufin.co.ke/terms` | Terms | Terms and conditions |
| `https://edufin.co.ke/privacy` | Privacy Policy | Privacy policy |

### 1.2 WordPress Admin

| URL | Purpose | Access |
|-----|---------|--------|
| `https://edufin.co.ke/wp-admin` | WordPress admin dashboard | Secretarial staff / IT admin |
| `https://edufin.co.ke/wp-login.php` | WordPress login | Staff only |

### 1.3 Cross-Links to Laravel Application

These are standard public hyperlinks rendered in the WordPress header (no authentication handoff):

| Link Label | Target URL | Purpose |
|------------|------------|---------|
| Login | `https://app.edufin.co.ke/login` | Client/staff login (role-based redirect after auth) |
| Get Started | `https://app.edufin.co.ke/register` | New client onboarding / registration |
| My Portal | `https://app.edufin.co.ke/dashboard` | Client dashboard (when logged in) |

---

## 2. Laravel Site (app.edufin.co.ke)

### 2.1 Authentication & Onboarding

| URL | Page | Auth | Description |
|-----|------|------|-------------|
| `https://app.edufin.co.ke/login` | Login | Public | Unified login for all users (clients and staff) |
| `https://app.edufin.co.ke/register` | Register | Public | New client onboarding / registration |
| `https://app.edufin.co.ke/forgot-password` | Forgot Password | Public | Password reset request |
| `https://app.edufin.co.ke/logout` | Logout | Authenticated | End session |

#### Role-Based Redirection After Login

After a successful login at `app.edufin.co.ke/login`, the system redirects users to their respective dashboards based on their assigned roles:

| Role | Redirect Destination | Description |
|------|---------------------|-------------|
| Client | `https://app.edufin.co.ke/dashboard` | Client self-service portal |
| Loan Officer | `https://app.edufin.co.ke/admin` | Staff admin panel (Filament) |
| KYC Verifier | `https://app.edufin.co.ke/admin` | Staff admin panel (Filament) |
| System Administrator | `https://app.edufin.co.ke/admin` | Staff admin panel (Filament) |
| Super Admin | `https://app.edufin.co.ke/admin` | Full system access (Filament) |

> **Note:** The `admin.edufin.co.ke` subdomain has been removed. The admin panel (Filament) is now served at the path `app.edufin.co.ke/admin`. All staff roles are redirected there after login.

### 2.2 Client Portal (Web - Livewire)

| URL | Page | Auth | Description |
|-----|------|------|-------------|
| `https://app.edufin.co.ke/` | Landing / redirect | Public | Redirects to login or dashboard |
| `https://app.edufin.co.ke/dashboard` | Dashboard | Authenticated (Client) | Account overview |
| `https://app.edufin.co.ke/profile` | Profile | Authenticated | Profile management |
| `https://app.edufin.co.ke/profile/kyc` | KYC | Authenticated | KYC document submission |
| `https://app.edufin.co.ke/loans` | Loans | Authenticated | List of user's loans |
| `https://app.edufin.co.ke/loans/apply` | Apply | Authenticated | Loan application workflow |
| `https://app.edufin.co.ke/loans/{id}` | Loan Details | Authenticated | Individual loan view |
| `https://app.edufin.co.ke/loans/{id}/schedule` | Schedule | Authenticated | Payment schedule |
| `https://app.edufin.co.ke/loans/{id}/statement` | Statement | Authenticated | Generate / view statement |
| `https://app.edufin.co.ke/beneficiaries` | Beneficiaries | Authenticated | Education beneficiaries |
| `https://app.edufin.co.ke/beneficiaries/{id}` | Beneficiary Details | Authenticated | Individual beneficiary |
| `https://app.edufin.co.ke/collateral` | Collateral | Authenticated | Collateral records |
| `https://app.edufin.co.ke/collateral/{id}` | Collateral Details | Authenticated | Individual collateral |
| `https://app.edufin.co.ke/documents` | Documents | Authenticated | Document list |
| `https://app.edufin.co.ke/documents/upload` | Upload | Authenticated | Document upload |
| `https://app.edufin.co.ke/notifications` | Notifications | Authenticated | Notification center |
| `https://app.edufin.co.ke/payments` | Payments | Authenticated | Payment history |

### 2.3 Admin Panel (app.edufin.co.ke/admin - Filament)

> The admin panel was previously served on the `admin.edufin.co.ke` subdomain. It is now served at the path `app.edufin.co.ke/admin` on the Laravel application domain. Staff access it by logging in at `app.edufin.co.ke/login` and being redirected based on their role.

| URL | Page | Access | Description |
|-----|------|--------|-------------|
| `https://app.edufin.co.ke/admin` | Admin Dashboard | Staff | Admin home |
| `https://app.edufin.co.ke/admin/login` | Admin Login | Staff | Staff login (redirects to /login) |
| `https://app.edufin.co.ke/admin/users` | Users | Staff | User & role management |
| `https://app.edufin.co.ke/admin/loans` | Loans | Staff | Loan application review queue |
| `https://app.edufin.co.ke/admin/kyc` | KYC | Staff | KYC verification workflow |
| `https://app.edufin.co.ke/admin/collateral` | Collateral | Staff | Collateral approval |
| `https://app.edufin.co.ke/admin/disbursements` | Disbursements | Staff | Disbursement processing |
| `https://app.edufin.co.ke/admin/reports` | Reports | Staff | Reports & analytics |
| `https://app.edufin.co.ke/admin/audit-logs` | Audit Logs | Staff | Audit log viewer |
| `https://app.edufin.co.ke/admin/settings` | Settings | Staff | System configuration |

### 2.4 REST API (edufin.co.ke/api/v1)

**Base URL:** `https://edufin.co.ke/api/v1`

> The API was previously served on the `api.edufin.co.ke` subdomain. It is now served via a path-based prefix on the main domain. Nginx routes `/api/` requests on `edufin.co.ke` to the Laravel application's `public/index.php`.

#### Public Endpoints (API Key: `X-API-Key`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `https://edufin.co.ke/api/v1/packages` | List financing packages |
| GET | `https://edufin.co.ke/api/v1/packages/{slug}` | Package details |
| GET | `https://edufin.co.ke/api/v1/calculator/rates` | Loan calculator rates |
| POST | `https://edufin.co.ke/api/v1/inquiries` | Submit contact inquiry |
| POST | `https://edufin.co.ke/api/v1/newsletter/subscribe` | Newsletter signup |

#### Authentication Endpoints (Public)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `https://edufin.co.ke/api/v1/auth/register` | New user registration |
| POST | `https://edufin.co.ke/api/v1/auth/login` | Login (returns JWT) |
| POST | `https://edufin.co.ke/api/v1/auth/refresh` | Refresh JWT token |
| POST | `https://edufin.co.ke/api/v1/auth/logout` | Logout |
| POST | `https://edufin.co.ke/api/v1/auth/verify-otp` | Verify phone/email OTP |
| POST | `https://edufin.co.ke/api/v1/auth/forgot-password` | Password reset request |

#### Authenticated Endpoints (JWT: `Authorization: Bearer {token}`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `https://edufin.co.ke/api/v1/profile` | Get user profile |
| PUT | `https://edufin.co.ke/api/v1/profile` | Update profile |
| POST | `https://edufin.co.ke/api/v1/profile/kyc` | Submit KYC documents |
| GET | `https://edufin.co.ke/api/v1/profile/kyc/status` | KYC verification status |
| GET | `https://edufin.co.ke/api/v1/beneficiaries` | List beneficiaries |
| POST | `https://edufin.co.ke/api/v1/beneficiaries` | Add beneficiary |
| GET | `https://edufin.co.ke/api/v1/beneficiaries/{id}` | Beneficiary details |
| PUT | `https://edufin.co.ke/api/v1/beneficiaries/{id}` | Update beneficiary |
| GET | `https://edufin.co.ke/api/v1/loans` | List user's loans |
| POST | `https://edufin.co.ke/api/v1/loans` | Apply for loan |
| GET | `https://edufin.co.ke/api/v1/loans/{id}` | Loan details |
| GET | `https://edufin.co.ke/api/v1/loans/{id}/schedule` | Payment schedule |
| GET | `https://edufin.co.ke/api/v1/loans/{id}/statement` | Generate statement |
| GET | `https://edufin.co.ke/api/v1/collateral` | List collateral |
| POST | `https://edufin.co.ke/api/v1/collateral` | Register collateral |
| GET | `https://edufin.co.ke/api/v1/collateral/{id}` | Collateral details |
| POST | `https://edufin.co.ke/api/v1/documents/upload` | Upload document (signed URL) |
| GET | `https://edufin.co.ke/api/v1/documents` | List documents |
| GET | `https://edufin.co.ke/api/v1/documents/{id}` | Download document (signed URL) |
| GET | `https://edufin.co.ke/api/v1/notifications` | List notifications |
| PUT | `https://edufin.co.ke/api/v1/notifications/{id}` | Mark as read |
| PUT | `https://edufin.co.ke/api/v1/notifications/read-all` | Mark all as read |

#### Webhook Endpoints (Signature: `X-Webhook-Signature`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `https://edufin.co.ke/api/v1/webhooks/banking/payment` | Payment notification from CBS |
| POST | `https://edufin.co.ke/api/v1/webhooks/banking/status` | Loan status update from CBS |
| POST | `https://edufin.co.ke/api/v1/webhooks/mpesa/callback` | M-Pesa payment callback |
| POST | `https://edufin.co.ke/api/v1/webhooks/sms/delivery` | SMS delivery report |

#### Health Check

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `https://edufin.co.ke/api/v1/health` | Service health (database, redis, queue) |

### 2.5 Rate Limits

| Endpoint Type | Limit |
|---------------|-------|
| Public (API Key) | 100 req/min |
| Authenticated (JWT) | 60 req/min per user |
| Webhooks | Unlimited |

---

## 3. Static Assets (cdn.edufin.co.ke)

| URL | Type | Description |
|-----|------|-------------|
| `https://cdn.edufin.co.ke/images/logo.svg` | Image | EduFin logo |
| `https://cdn.edufin.co.ke/css/variables.css` | CSS | Shared design tokens |
| `https://cdn.edufin.co.ke/images/` | Image | Media library |
| `https://cdn.edufin.co.ke/content/` | Mixed | WordPress wp-content (media, uploads) |

---

## 4. Email & DNS Records

| Record | Type | Value | Purpose |
|--------|------|-------|---------|
| `edufin.co.ke` | A | `<WP_SERVER_IP>` | WordPress + API path routing (proxied) |
| `www.edufin.co.ke` | CNAME | `edufin.co.ke` | WordPress redirect |
| `app.edufin.co.ke` | A | `<LARAVEL_SERVER_IP>` | Laravel portal + admin (proxied) |
| `cdn.edufin.co.ke` | CNAME | `<R2_BUCKET>.r2.dev` | Static assets (proxied) |
| `mail.edufin.co.ke` | MX | `10 mail.edufin.co.ke` | Email |
| `edufin.co.ke` | TXT | `v=spf1 include:_spf.google.com ~all` | SPF |

> **Note:** The `api.edufin.co.ke` and `admin.edufin.co.ke` DNS records have been removed. The API is served via path-based routing at `edufin.co.ke/api/v1/...`, and the admin panel is served at `app.edufin.co.ke/admin`.

---

## 5. Nginx Routing Summary

```
edufin.co.ke
├── /                          → WordPress (PHP-FPM 8.2)
├── /products, /blog, /about   → WordPress (PHP-FPM 8.2)
├── /wp-admin, /wp-login.php   → WordPress (PHP-FPM 8.2)
└── /api/                      → Laravel (PHP-FPM 8.3) [reverse proxy]

app.edufin.co.ke
├── /login                     → Laravel (auth, role-based redirect)
├── /register                  → Laravel (onboarding)
├── /dashboard                 → Laravel (client portal, Livewire)
├── /admin                     → Laravel (Filament admin panel)
├── /profile, /loans, /kyc     → Laravel (client portal, Livewire)
└── /api/v1/*                  → Laravel (REST API, same app)

cdn.edufin.co.ke
└── /                          → Cloudflare R2 (static assets)
```

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Technical Team | Initial site links reference |
| 1.1 | 2026-08-06 | EduFin Technical Team | API moved to edufin.co.ke/api/v1 path-based routing; admin.edufin.co.ke removed (admin at app.edufin.co.ke/admin); login at app.edufin.co.ke/login with role-based redirect; onboarding at app.edufin.co.ke/register |
