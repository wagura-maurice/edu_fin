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
│  • Links to Laravel portal for login & registration                            │
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
│  • Register button → app.edufin.co.ke/register                                 │
│  • Standard HTML links (no API or SSO integration)                             │
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

WordPress does not use custom plugins for Laravel integration. Login and registration buttons on the WordPress site link directly to the Laravel portal via standard HTML links:

- **Login:** `app.edufin.co.ke/login`
- **Register:** `app.edufin.co.ke/register`

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
