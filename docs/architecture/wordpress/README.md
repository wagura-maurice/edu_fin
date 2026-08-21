# WordPress Architecture

## Company Landing Page

**Version:** 2.0  
**Last Updated:** August 10, 2026

---

## Overview

WordPress serves as the **standalone company landing page**, completely decoupled from the core application logic. It is managed internally by company secretarial staff and contains NO business data or PII.

## Role Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        WORDPRESS - LANDING PAGE ONLY                            │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PURPOSE:                                                                       │
│  • Public-facing brand presence                                                │
│  • Marketing content management                                                │
│  • SEO optimization                                                            │
│  • Customer support chat widget (bottom-right corner)                          │
  • Client onboarding wizard (/get-started) — submits to Laravel API            │
  • Public loan tracking (/track-application) — consumes Laravel tracking API  │
  • Links to Laravel portal for login (registration via API, not a link)        │
│                                                                                 │
│  MANAGED BY:                                                                   │
│  • Secretarial staff (content)                                                 │
│  • IT Admin (plugins, security)                                                │
│                                                                                 │
│  DOES NOT CONTAIN:                                                             │
│  ✗ Client account data                                                         │
│  ✗ Loan information                                                            │
│  ✗ KYC documents                                                               │
│  ✗ Financial transactions                                                      │
│  ✗ Any PII beyond basic WP users                                              │
│                                                                                 │
│  LINKS TO LARAVEL:                                                             │
│  • Login button → app.edufin.co.ke/login                                       │
  • Onboarding wizard → edufin.co.ke/get-started (submits to Laravel API)       │
  • Loan tracking → edufin.co.ke/track-application (consumes Laravel tracking API) │
  • Consumes Laravel API: POST /api/v1/auth/register + GET /api/v1/options/*    │
  • Consumes Laravel API: GET /api/v1/loans/track/{reference}                  │
  • No SSO, no shared sessions (API contract only)                              │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Content Managed

| Content Type | Description | Update Frequency |
|--------------|-------------|------------------|
| Company Profile | About, mission, team | Monthly |
| Product Catalog | Financing packages | As needed |
| Blog/News | Articles, announcements | Weekly |
| FAQs | Help content | As needed |
| Testimonials | Client reviews | Monthly |
| Legal Documents | Terms & Conditions, Privacy Policy, Data Protection, Cookie Policy, Refund & Cancellation, Complaints & Disputes, Regulatory & Licensing | Annually |
| Contact Info | Addresses, phones | As needed |
| Support Chat Widget | Bottom-right chat widget (powered by Support Agent) | Always active |

## Technology Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| WordPress | 6.x | CMS platform |
| PHP | 8.2 | Runtime |
| MySQL | 8.0 | Content database |
| Redis | 7.x | Object cache |
| Nginx | 1.24 | Web server |

## Custom Plugins

WordPress does not use custom plugins for Laravel integration. The login button links directly to the Laravel portal. Registration is handled by the onboarding wizard, and loan status tracking is handled by the tracking page — both consume the Laravel REST API:

- **Login (link):** `app.edufin.co.ke/login`
- **Onboarding wizard (WordPress page):** `edufin.co.ke/get-started`
  - Fetches dynamic options: `GET edufin.co.ke/api/v1/options/*`
  - Submits registration: `POST edufin.co.ke/api/v1/auth/register`
- **Loan tracking (WordPress page):** `edufin.co.ke/track-application`
  - Looks up status: `GET edufin.co.ke/api/v1/loans/track/{reference}`
  - Returns only whitelisted public fields (no financial amounts or PII)
  - See [Public Loan Tracking](../../features/public-loan-tracking.md) for full specification

> **Note:** During the current development phase, the wizard uses mock/demo REST API calls to simulate the Laravel backend integration. The mock endpoints are served by WordPress itself (via WordPress REST API routes) and return the same JSON shape that the real Laravel endpoints will return. Switching to the real Laravel backend only requires changing the API base URL.

## Approved Third-Party Plugins

| Plugin | Purpose | Required |
|--------|---------|----------|
| Yoast SEO | SEO optimization | Yes |
| WP Rocket | Performance caching | Yes |
| Wordfence | Security | Yes |
| UpdraftPlus | Backups | Yes |
| Mailchimp | Newsletter | Optional |

## Support Chat Widget

The WordPress site includes a **support chat widget** embedded in the bottom-right corner of every page. This widget is powered by the **Support Agent** (part of the EduFin AI Agents system) and provides real-time customer support to website visitors.

### Widget Behavior

| Feature | Description |
|---------|-------------|
| **Location** | Bottom-right corner of all WordPress pages |
| **Visibility** | Always visible; shows online/offline status |
| **Authentication** | Anonymous (no login required); collects name/email/phone for follow-up |
| **Real-time chat** | When Support Agent is online, visitors chat in real-time via WebSocket |
| **Offline mode** | When agent is offline, displays a contact form to collect visitor info for follow-up |
| **Rate limiting** | Max 10 messages per minute per visitor (abuse prevention) |
| **Escalation** | Complex queries are escalated to human support staff; visitor is notified |

### Technical Integration

| Aspect | Specification |
|--------|---------------|
| **Embedding** | JavaScript snippet loaded on all WordPress pages (via theme `functions.php` or header plugin) |
| **Communication** | WebSocket connection to Support Agent backend |
| **Agent** | Support Agent (MCP Server) — see [Support Agent Architecture](../ai-agents/support-agent.md) |
| **Channels** | Chat widget, WhatsApp (via WAHA), Email (customer_care@edufin.co.ke, support@edufin.co.ke) |
| **Data stored** | Conversation transcripts, visitor contact info (if provided) — NO PII from Laravel |

> **Note:** The support chat widget does NOT access Laravel business data (loans, clients, KYC). It can answer FAQs about EduFin products and direct visitors to the Laravel portal for account-specific actions. The widget is managed by the AI Agents layer and does not require a WordPress plugin — it is a lightweight JavaScript embed that connects directly to the Support Agent backend.

## Client Onboarding Wizard

The WordPress site hosts a multi-step "Get Started" onboarding wizard at `edufin.co.ke/get-started`. This wizard is the primary client registration interface — it collects client data across multiple steps and submits it to the Laravel registration API.

### Wizard Steps

| Step | Title | Fields |
|------|-------|--------|
| 1 | Personal Details | Full name, email, phone, gender (dynamic), location/county (dynamic) |
| 2 | Employment & Income | Employment type (dynamic), employer name, monthly income range (dynamic) |
| 3 | Education Beneficiary | Beneficiary name, relationship (dynamic), school name, education level (dynamic) |
| 4 | Account Setup | Password, confirm password, terms agreement |
| 5 | Review & Confirm | Summary of all entered data — user confirms before submission |

### Dynamic Options

Dropdown fields (gender, location, employment type, income range, education level, relationship type) are populated via API calls to Laravel's `/options/*` endpoints. This ensures the wizard always reflects the latest backend data without WordPress code changes.

### Mock API (Current Phase)

During the current development phase, the wizard uses **mock/demo REST API calls** to simulate the Laravel backend. The mock endpoints are registered as WordPress REST routes (`/wp-json/edufin/v1/options/*`) and return the same JSON shape as the future Laravel endpoints. Switching to production only requires updating the API base URL configuration.

### Data Flow

```
WordPress Wizard                Laravel API
─────────────────               ─────────────────
Load wizard
  │
  ├── GET /options/* ──────────► Returns JSON option lists
  │     ◄──────────────────────  { locations, genders, ... }
  │
  User completes steps 1–4
  │
  Step 5: Review & confirm
  │
  └── POST /auth/register ─────► Validates + creates user
        ◄──────────────────────  { token, account_id }
  │
  Show success screen
  │
  └── Link to app.edufin.co.ke/login
```

> **Important:** The WordPress wizard collects and forwards data but does **not** persist PII. All data storage happens in Laravel. The wizard only retains data in the browser (JavaScript state) during the wizard session and discards it after submission.

## Public Loan Tracking

The WordPress site hosts a public "Track Application" page at `edufin.co.ke/track-application`. This page allows anyone — applicants, obligors, or third parties — to check the status of a loan application by entering a tracking number, **without logging in to the Laravel portal**.

### How It Works

1. User visits the "Track Application" page on the WordPress site
2. User enters their tracking number (format: `EDF-YYYY-XXXXXX`) and passes Turnstile verification
3. JavaScript calls `GET edufin.co.ke/api/v1/loans/track/{reference}` with the public API key
4. Laravel looks up the loan facility and returns a **whitelisted** set of non-sensitive fields
5. JavaScript renders a status card showing: status badge, package name, beneficiary initials, application date, last updated date, and a link to the portal for full details

### Data Classification

The tracking endpoint returns **only** non-sensitive public fields:

| Field | Example | Sensitive? |
|-------|---------|------------|
| `reference` | `EDF-2026-08X7K2` | No (user already knows this) |
| `status` / `status_label` | `ACTIVE` / "Active" | No (status enum only) |
| `package_name` | "University Financing" | No (public on /products page) |
| `beneficiary_initials` | "John D." | No (first name + last initial only) |
| `application_date` | "2026-07-15" | No (date only) |
| `last_updated` | "2026-08-01" | No (date only) |

All financial amounts (principal, interest, balance), personal information (full names, email, phone), collateral details, and document URLs remain in the authenticated Laravel portal only.

### Mock API (Current Phase)

During the current development phase, the tracking page uses a mock WordPress REST endpoint (`/wp-json/edufin/v1/loans/track/{reference}`) that returns the same JSON shape as the future Laravel endpoint. Switching to production only requires updating the API base URL configuration — identical to the onboarding wizard migration path.

> **Full specification:** See [Public Loan Tracking](../../features/public-loan-tracking.md) for the complete technical specification, including API contract, security controls, data classification, and implementation phases.

## Security Configuration

```php
// wp-config.php security settings

define('DISALLOW_FILE_EDIT', true);    // No theme/plugin editing
define('DISALLOW_FILE_MODS', true);    // No plugin installation
define('FORCE_SSL_ADMIN', true);       // HTTPS for admin
$table_prefix = 'edf_';                // Non-default prefix
```

## Staff Access Levels

| Role | Capabilities |
|------|--------------|
| **Secretarial Staff** | Edit pages, posts, FAQs, testimonials, media |
| **IT Admin** | Plugin updates, security, settings |

**Restrictions for Secretarial Staff:**
- Cannot install/modify plugins
- Cannot access theme editor
- Cannot modify settings
- Cannot manage users

---

**See Also:**
- [Content Management Guide](./content-management.md)
- [Theme Structure](./theme-structure.md)
- [Plugin Documentation](./plugins.md)
- [AI Agents Architecture](../ai-agents/README.md)
- [Support Agent](../ai-agents/support-agent.md)
