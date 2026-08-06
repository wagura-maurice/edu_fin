# WordPress Architecture

## Company Landing Page

**Version:** 1.0  
**Last Updated:** August 6, 2026

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
│  • Lead generation (forms submit to Laravel)                                   │
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
│  INTEGRATION WITH LARAVEL:                                                     │
│  • SSO authentication (redirect to Laravel)                                    │
│  • Public data fetch (packages, rates)                                         │
│  • Form submissions (inquiries to Laravel API)                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Content Managed

| Content Type | Description | Update Frequency |
|--------------|-------------|------------------|
| Company Profile | About, mission, team | Monthly |
| Product Catalog | Financing packages (from API) | Real-time |
| Blog/News | Articles, announcements | Weekly |
| FAQs | Help content | As needed |
| Testimonials | Client reviews | Monthly |
| Legal Documents | Terms, Privacy Policy | Annually |
| Contact Info | Addresses, phones | As needed |

## Technology Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| WordPress | 6.x | CMS platform |
| PHP | 8.2 | Runtime |
| MySQL | 8.0 | Content database |
| Redis | 7.x | Object cache |
| Nginx | 1.24 | Web server |

## Custom Plugins

### EduFin SSO (`edufin-sso`)
Handles Single Sign-On with Laravel portal.

**Features:**
- Redirects login to Laravel SSO
- Validates SSO tokens from Laravel
- Creates WordPress sessions for authenticated users
- Handles logout synchronization

### EduFin API (`edufin-api`)
Fetches public data from Laravel API.

**Features:**
- Caches API responses
- Provides shortcodes for dynamic content
- Submits contact forms to Laravel

**Shortcodes:**
- `[edufin_products]` - Display financing packages
- `[edufin_calculator]` - Loan calculator widget
- `[edufin_login_button]` - Login/portal button

## Approved Third-Party Plugins

| Plugin | Purpose | Required |
|--------|---------|----------|
| Yoast SEO | SEO optimization | Yes |
| WP Rocket | Performance caching | Yes |
| Wordfence | Security | Yes |
| UpdraftPlus | Backups | Yes |
| Tawk.to | Live chat | Optional |
| Mailchimp | Newsletter | Optional |

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
