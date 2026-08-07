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
│  • Customer support chat widget (bottom-right corner)                          │
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
