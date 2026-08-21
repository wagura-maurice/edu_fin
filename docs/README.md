# EduFin Documentation

## Documentation Structure

```
docs/
├── README.md                           # This file - Documentation index
├── project_documentation.md            # Global project overview
├── implementation.md                   # Technical implementation details
│
├── architecture/                       # Architecture documentation
│   ├── overview.md                     # System architecture overview
│   ├── wordpress/                      # WordPress (Landing Page)
│   │   ├── README.md                   # WordPress architecture overview
│   │   ├── content-management.md       # Content management guide
│   │   ├── theme-structure.md          # Theme architecture
│   │   └── plugins.md                  # Plugin documentation
│   │
│   └── laravel/                        # Laravel (Core Application)
│       ├── README.md                   # Laravel architecture overview
│       ├── portal.md                   # Client & Admin portal
│       ├── api.md                      # API layer documentation
│       ├── services.md                 # Business services
│       └── banking-integration.md      # Core banking integration
│
│   ai-agents/                          # AI Agents (Agentic Intelligence Layer)
│       ├── README.md                   # AI Agents architecture overview
│       ├── master-agent.md             # Master Agent orchestration & HITL
│       ├── mcp-protocol.md             # MCP protocol specification
│       ├── marketing-agent.md          # Marketing sub-agent (social media, SEO, email, WhatsApp)
│       ├── email-agent.md              # Email & communication sub-agent
│       ├── support-agent.md            # Support sub-agent (chat widget, WhatsApp, email support)
│       ├── self-healing-agent.md       # Self-healing sub-agent
│       └── integration.md              # Technical integration & workflow
│
├── api/                                # API documentation
│   ├── README.md                       # API overview
│   ├── authentication.md               # Auth endpoints
│   ├── clients.md                      # Client endpoints
│   ├── loans.md                        # Loan endpoints (incl. public tracking)
│   └── webhooks.md                     # Webhook specifications
│
├── features/                           # Feature specifications
│   └── public-loan-tracking.md         # Public loan tracking (WordPress UI + Laravel API)
│
├── security/                           # Security documentation
│   ├── README.md                       # Security overview
│   ├── authentication.md               # Auth & Role-Based Access Control
│   ├── authorization.md                # RBAC & permissions
│   └── data-protection.md              # Encryption & compliance
│
├── deployment/                         # Deployment documentation
│   ├── README.md                       # Deployment overview
│   ├── infrastructure.md               # Infrastructure setup
│   ├── wordpress.md                    # WordPress deployment
│   └── laravel.md                      # Laravel deployment
│
└── guides/                             # User & developer guides
    ├── getting-started.md              # Quick start guide
    ├── secretarial-staff.md            # WordPress admin guide
    └── developer-setup.md              # Development environment
```

## Quick Links

### For Business Stakeholders
- [Project Documentation](./project_documentation.md) - Complete project overview
- [Architecture Overview](./architecture/overview.md) - System design

### For Developers
- [Implementation Guide](./implementation.md) - Technical implementation
- [API Documentation](./api/README.md) - API reference
- [Developer Setup](./guides/developer-setup.md) - Development environment
- [AI Agents Architecture](./architecture/ai-agents/README.md) - Agentic intelligence layer
- [Public Loan Tracking](./features/public-loan-tracking.md) - WordPress tracking page + Laravel API spec

### For Operations
- [Deployment Guide](./deployment/README.md) - Infrastructure & deployment
- [Security Documentation](./security/README.md) - Security protocols

### For Content Administrators
- [Secretarial Staff Guide](./guides/secretarial-staff.md) - WordPress content management

---

## Document Versioning

| Document | Version | Last Updated |
|----------|---------|--------------|
| Project Documentation | 1.0 | 2026-08-06 |
| Implementation Guide | 1.0 | 2026-08-06 |
| Architecture Docs | 1.0 | 2026-08-06 |
| AI Agents Architecture | 1.0 | 2026-08-08 |

