# EduFin - Site Links Reference

**Version:** 2.0  
**Last Updated:** August 10, 2026

---

## Overview

EduFin operates as two separate systems with a clearly defined API contract between them. WordPress is the public-facing marketing site **and the client onboarding interface**; Laravel is the core application engine that exposes a REST API consumed by WordPress (for onboarding) and by future mobile applications.

| System | Domain | Technology | Purpose |
|--------|--------|------------|---------|
| Public Website + Onboarding | `edufin.co.ke` | WordPress | Marketing, SEO, blog, content, **client onboarding wizard** |
| Application | `app.edufin.co.ke` | Laravel (Livewire) | Client portal, admin panel, KYC, loans, statements |
| REST API | `edufin.co.ke/api/v1` | Laravel | Registration, dynamic options, mobile app, webhooks, integrations (path-based on main domain) |
| Static Assets | `cdn.edufin.co.ke` | Cloudflare R2 | Media, images, CSS, JS |

> **Note:** The two systems are connected by a REST API contract. WordPress consumes Laravel's registration and dynamic-options endpoints to power the onboarding wizard. WordPress links to the Laravel application for login via standard public HTTP links. There is no SSO and no shared sessions.
>
> **API Routing:** The REST API is served via a path-based prefix on the main domain (`edufin.co.ke/api/v1/...`) rather than a dedicated API subdomain. Nginx routes `/api/` requests on `edufin.co.ke` to the Laravel application.
>
> **Authentication:** All users (clients and staff) access the login interface at `app.edufin.co.ke/login`. **Registration/onboarding no longer occurs on the Laravel UI.** Instead, the WordPress site hosts an onboarding wizard (`edufin.co.ke/get-started`) that collects client data and submits it to the Laravel registration API (`edufin.co.ke/api/v1/auth/register`). After a successful login, the system redirects users to their respective dashboards based on their assigned roles.

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
| `https://edufin.co.ke/get-started` | Get Started (Onboarding) | Multi-step client onboarding wizard |
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

These are hyperlinks rendered in the WordPress header. The "Get Started" link now points to the **WordPress onboarding wizard** (not the Laravel UI), since registration is handled via the API:

| Link Label | Target URL | Purpose |
|------------|------------|---------|
| Login | `https://app.edufin.co.ke/login` | Client/staff login (role-based redirect after auth) |
| Get Started | `https://edufin.co.ke/get-started` | New client onboarding wizard (WordPress; submits to Laravel API) |
| Client Portal | `https://app.edufin.co.ke/login` | External application login (footer + topbar link) |

---

## 2. Laravel Site (app.edufin.co.ke)

### 2.1 Authentication & Onboarding

> **Important:** The Laravel application does **not** host a front-end `/register` page. Registration is handled exclusively via the REST API (`edufin.co.ke/api/v1/auth/register`), which is consumed by the WordPress onboarding wizard and future mobile applications. The Laravel UI only provides `/login`, `/forgot-password`, and the standard password reset mechanisms.

| URL | Page | Auth | Description |
|-----|------|------|-------------|
| `https://app.edufin.co.ke/login` | Login | Public | Unified login for all users (clients and staff) |
| `https://app.edufin.co.ke/forgot-password` | Forgot Password | Public | Password reset request |
| `https://app.edufin.co.ke/password/reset` | Password Reset | Public | Password reset form (email link landing) |
| `https://app.edufin.co.ke/logout` | Logout | Authenticated | End session |

> **Registration (API-only):** `POST https://edufin.co.ke/api/v1/auth/register` — no web page. The WordPress onboarding wizard at `edufin.co.ke/get-started` collects client data across multiple steps and submits to this endpoint. See [Registration Flow](#registration-flow) below.

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

### 2.1.1 Registration Flow

The registration process is split across the two platforms: **WordPress** acts as the onboarding interface (UI), and **Laravel** manages the data logic and dynamic options (API).

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                          REGISTRATION / ONBOARDING FLOW                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│  WORDPRESS (edufin.co.ke/get-started)            LARAVEL API (edufin.co.ke/api/v1)  │
│  ─────────────────────────────────────            ─────────────────────────────────  │
│                                                                                     │
│  1. User visits the "Get Started" onboarding wizard                                 │
│     on the WordPress site.                                                          │
│                                                                                     │
│  2. Wizard loads ──────────────────────────────►  GET /options/locations            │
│     dynamic dropdown options                       GET /options/genders              │
│     (Location, Gender, Employment, etc.)          GET /options/employment-types     │
│     via API calls ◄──────────────────────────────  (returns JSON option lists)      │
│                                                                                     │
│  3. User completes multiple wizard steps:                                           │
│     Step 1: Personal details (name, email, phone, gender, location)                 │
│     Step 2: Employment & income (employment type, income range)                     │
│     Step 3: Education beneficiary (school, level, relationship)                     │
│     Step 4: Account credentials (password, confirm)                                 │
│     Step 5: Review & confirm (summary of all entered data)                          │
│                                                                                     │
│  4. User confirms ──────────────────────────────►  POST /auth/register              │
│     submission                                     (validates, creates user,         │
│     ◄───────────────────────────────────────────   returns JWT + account ID)        │
│                                                                                     │
│  5. WordPress shows success screen with a                                          │
│     "Proceed to Login" link ─────────────────────►  app.edufin.co.ke/login          │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

**Key principles:**

- **WordPress** owns the onboarding **UI** — the multi-step wizard, form validation (client-side), and user experience. It does **not** store business data; it only collects and forwards it.
- **Laravel** owns the onboarding **data logic** — server-side validation, user creation, dynamic option sources (locations, genders, employment types, etc.), and the registration business rules.
- **Dynamic options** (dropdowns for Location, Gender, Employment Type, Income Range, Education Level, Relationship Type) are fetched from Laravel's `/options/*` endpoints at wizard load time, so the WordPress wizard always reflects the latest backend data without code changes.
- The registration API is designed to be **consumer-agnostic**: the WordPress site is the primary consumer today, and future mobile applications (Flutter) will consume the same `POST /auth/register` endpoint.
- The Laravel UI intentionally has **no `/register` page** — this prevents a duplicate onboarding surface and keeps all registration logic centralized in the API layer.

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
| GET | `https://edufin.co.ke/api/v1/options/locations` | List locations (counties/cities) for onboarding dropdowns |
| GET | `https://edufin.co.ke/api/v1/options/genders` | List gender options for onboarding dropdowns |
| GET | `https://edufin.co.ke/api/v1/options/employment-types` | List employment types for onboarding dropdowns |
| GET | `https://edufin.co.ke/api/v1/options/income-ranges` | List income ranges for onboarding dropdowns |
| GET | `https://edufin.co.ke/api/v1/options/education-levels` | List education levels for onboarding dropdowns |
| GET | `https://edufin.co.ke/api/v1/options/relationship-types` | List relationship types (beneficiary) for onboarding dropdowns |
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
| POST | `https://edufin.co.ke/api/v1/webhooks/whatsapp/delivery` | WhatsApp delivery report |

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
├── /get-started               → WordPress (onboarding wizard; submits to Laravel API)
├── /products, /blog, /about   → WordPress (PHP-FPM 8.2)
├── /wp-admin, /wp-login.php   → WordPress (PHP-FPM 8.2)
└── /api/                      → Laravel (PHP-FPM 8.3) [reverse proxy]

app.edufin.co.ke
├── /login                     → Laravel (auth, role-based redirect)
├── /forgot-password           → Laravel (password reset request)
├── /password/reset            → Laravel (password reset form)
├── /dashboard                 → Laravel (client portal, Livewire)
├── /admin                     → Laravel (Filament admin panel)
├── /profile, /loans, /kyc     → Laravel (client portal, Livewire)
└── /api/v1/*                  → Laravel (REST API, same app)

  NOTE: app.edufin.co.ke/register no longer exists.
        Registration is API-only: POST edufin.co.ke/api/v1/auth/register
        consumed by the WordPress onboarding wizard (edufin.co.ke/get-started).

cdn.edufin.co.ke
└── /                          → Cloudflare R2 (static assets)
```

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-08-06 | EduFin Technical Team | Initial site links reference |
| 1.1 | 2026-08-06 | EduFin Technical Team | API moved to edufin.co.ke/api/v1 path-based routing; admin.edufin.co.ke removed (admin at app.edufin.co.ke/admin); login at app.edufin.co.ke/login with role-based redirect; onboarding at app.edufin.co.ke/register |
| 2.0 | 2026-08-10 | EduFin Technical Team | Registration architecture change: Laravel no longer hosts a front-end /register page (API-only registration via POST /api/v1/auth/register). WordPress now hosts the onboarding wizard at edufin.co.ke/get-started. Added /options/* endpoints for dynamic dropdowns. Laravel UI limited to /login, /forgot-password, /password/reset. |
