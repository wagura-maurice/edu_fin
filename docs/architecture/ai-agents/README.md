# AI Agents Architecture

## Agentic Intelligence Layer

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Overview

The AI Agents module is an **autonomous intelligence layer** that sits alongside the existing EduFin dual-platform ecosystem (WordPress + Laravel). It is built on the **Model Context Protocol (MCP)** architecture and uses a hierarchical **Master Agent** pattern to orchestrate specialized sub-agents that perform marketing, communication, and infrastructure maintenance tasks.

This layer is **non-invasive**: it interacts with WordPress and Laravel through their existing APIs and interfaces without modifying core business logic. All agent actions are auditable, reversible where applicable, and subject to Human-in-the-Loop (HITL) oversight for high-stakes decisions.

## Role Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        AI AGENTS - INTELLIGENCE LAYER                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  PURPOSE:                                                                       │
│  • Autonomous marketing & content distribution (social media, email, WhatsApp) │
│  • Automated email communication & thread management                           │
│  • Customer support via chat widget, WhatsApp, and email                       │
│  • Self-healing infrastructure monitoring & code repair                        │
│  • Human-in-the-loop escalation for complex scenarios                          │
│                                                                                 │
│  ARCHITECTURE:                                                                  │
│  • Model Context Protocol (MCP) for inter-agent communication                  │
│  • Hierarchical Master Agent orchestration pattern                             │
│  • Each sub-agent is an MCP Server exposing scoped tools/resources             │
│  • Master Agent is an MCP Client/Host that routes and supervises              │
│                                                                                 │
│  INTERACTS WITH:                                                                │
│  • WordPress (REST API, wp-admin, chat widget) — content, marketing, support  │
│  • Laravel (REST API at edufin.co.ke/api/v1) — business data, notifications   │
│  • Social media (X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube)   │
│  • Email (SMTP/IMAP) — all EduFin mailboxes                                    │
│  • WhatsApp (WAHA server) — marketing broadcasts & customer support            │
│  • Git, deployment servers (self-healing)                                      │
│                                                                                 │
│  DOES NOT:                                                                      │
│  ✗ Modify Laravel business logic or financial transaction flows               │
│  ✗ Access PII or financial data without explicit HITL approval                │
│  ✗ Execute irreversible actions without management authorization              │
│  ✗ Bypass existing security layers (Cloudflare WAF, RBAC, auth)              │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              EDUFIN AI AGENTS ECOSYSTEM                              │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                         ┌─────────────────────┐                                    │
│                         │   MANAGEMENT /      │                                    │
│                         │   HITL INTERFACE    │                                    │
│                         │   (Dashboard,       │                                    │
│                         │    Slack, Email)    │                                    │
│                         └─────────┬───────────┘                                    │
│                                   │ Approve / Reject / Override                     │
│                                   ▼                                                 │
│                         ┌─────────────────────┐                                    │
│                         │                     │   Orchestration                    │
│                         │   MASTER AGENT      │   Task Routing                     │
│                         │   (MCP Host)        │   Result Aggregation               │
│                         │                     │   HITL Detection                   │
│                         │                     │   Escalation Management            │
│                         └──┬───┬──┬───┬──────┘                                    │
│                            │   │  │   │                                            │
│             MCP Protocol   │   │  │   │   MCP Protocol                             │
│             (JSON-RPC)     │   │  │   │   (JSON-RPC)                               │
│                            ▼   ▼  ▼   ▼                                            │
│          ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐                      │
│          │         │ │         │ │         │ │             │                      │
│          │ MARKET- │ │ EMAIL   │ │ SUPPORT │ │ SELF-       │                      │
│          │ ING     │ │ AGENT   │ │ AGENT   │ │ HEALING     │                      │
│          │ AGENT   │ │         │ │         │ │ AGENT       │                      │
│          │ (MCP    │ │ (MCP    │ │ (MCP    │ │ (MCP        │                      │
│          │  Server)│ │  Server)│ │  Server)│ │  Server)    │                      │
│          └──┬───┬──┘ └────┬────┘ └──┬───┬──┘ └──────┬──────┘                      │
│             │   │         │         │   │           │                              │
│    ┌────────┘   │    ┌────┘    ┌────┘   └────┐      └──────────┐                  │
│    ▼            ▼    ▼         ▼             ▼                 ▼                  │
│  ┌──────┐  ┌──────┐ ┌──────┐ ┌──────┐  ┌──────────┐  ┌──────────────┐            │
│  │Social│  │ WP   │ │ SMTP │ │ WP   │  │ WAHA     │  │ Git Repo     │            │
│  │Media │  │ REST │ │ IMAP │ │ Chat │  │ WhatsApp │  │ WP/LV Code   │            │
│  │ APIs │  │ API  │ │ All  │ │Widget│  │ Server   │  │ Server Health│            │
│  │      │  │      │ │boxes │ │      │  │          │  │              │            │
│  └──────┘  └──┬───┘ └──────┘ └──────┘  └──────────┘  └──────┬───────┘            │
│              │                                              │                    │
│              │          ┌──────────────┐                    │                    │
│              └─────────►│   LARAVEL    │◄───────────────────┘                    │
│                         │   REST API   │                                         │
│                         │ edufin.co.ke │                                         │
│                         │  /api/v1     │                                         │
│                         └──────┬───────┘                                         │
│                                │                                                  │
│                                ▼                                                  │
│                         ┌──────────────┐                                         │
│                         │ POSTGRESQL   │                                         │
│                         │ (Audit Log   │                                         │
│                         │  Storage)    │                                         │
│                         └──────────────┘                                         │
│                                                                                     │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                              EXISTING EDUFIN PLATFORM                               │
│                                                                                     │
│   ┌─────────────────────┐              ┌─────────────────────────────────┐        │
│   │   WORDPRESS         │              │   LARAVEL                       │        │
│   │   edufin.co.ke      │              │   app.edufin.co.ke              │        │
│   │   (Content, Blog,   │              │   (Portal, Admin, API)          │        │
│   │    Chat Widget)     │              │                                 │        │
│   └─────────────────────┘              └─────────────────────────────────┘        │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

> **Social Media APIs:** X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube
> **EduFin Mailboxes:** info@, support@, marketing@, customer_care@, communications@edufin.co.ke

## Agent Inventory

| Agent | Type | MCP Role | Primary Domain | Key Capabilities |
|-------|------|----------|----------------|------------------|
| **Master Agent** | Orchestrator | MCP Host/Client | Cross-cutting | Task routing, result aggregation, HITL, escalation |
| **Marketing Agent** | Sub-Agent | MCP Server | Social Media + WordPress + Email + WhatsApp | Trend analysis, content generation, multi-platform posting, SEO suggestions, email campaigns, WhatsApp broadcasts |
| **Email Agent** | Sub-Agent | MCP Server | Email (SMTP/IMAP) | Conversation initiation, thread tracking, auto-response, multi-mailbox coordination |
| **Support Agent** | Sub-Agent | MCP Server | Customer Support | WordPress chat widget, WhatsApp support (WAHA), email support, FAQ responses, staff escalation |
| **Self-Healing Agent** | Sub-Agent | MCP Server | Infrastructure | Code monitoring, bug detection, patch generation, deployment |

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| MCP Protocol | JSON-RPC 2.0 over stdio/SSE | Inter-agent communication |
| Master Agent Runtime | Python / Node.js | Orchestration engine |
| Sub-Agent Runtimes | Python / Node.js | Specialized agent execution |
| LLM Integration | OpenAI / Anthropic API | Agent reasoning & generation |
| State Store | Redis | Agent state, task queues, conversation threads |
| Audit Log | PostgreSQL (Laravel DB) | Agent action audit trail |
| HITL Interface | Web Dashboard + Slack | Management approval workflow |
| Monitoring | Cloudflare + internal health checks | Agent health & uptime |

## MCP Architecture Summary

The system follows the **Model Context Protocol** specification:

| MCP Concept | EduFin Implementation |
|-------------|----------------------|
| **MCP Host** | Master Agent — consumes tools/resources from sub-agents |
| **MCP Client** | Master Agent's connection manager (1:1 per sub-agent) |
| **MCP Server** | Each sub-agent (Marketing, Email, Support, Self-Healing) |
| **Tools** | Scoped capabilities exposed by each sub-agent (e.g., `post_tweet`, `send_email`, `handle_chat_message`, `run_diagnostic`) |
| **Resources** | Read-only data sources (e.g., WordPress content, email threads, server logs) |
| **Prompts** | Pre-defined task templates for common agent workflows |

## Key Principles

1. **Non-Invasive Integration** - Agents interact via existing APIs; no core code modifications
2. **MCP-Native** - All inter-agent communication follows the MCP specification
3. **Hierarchical Control** - Master Agent supervises all sub-agents; no direct sub-agent-to-sub-agent communication
4. **Human-in-the-Loop** - High-stakes decisions require management approval before execution
5. **Full Auditability** - Every agent action is logged with reasoning, inputs, and outputs
6. **Graceful Degradation** - Agent failures do not affect the core EduFin platform
7. **Security Preservation** - Agents operate within existing RBAC and security boundaries

## Documentation Structure

```
docs/architecture/ai-agents/
├── README.md                   # This file - AI Agents overview
├── master-agent.md             # Master Agent architecture (orchestration + HITL)
├── mcp-protocol.md             # MCP protocol specification & communication contracts
├── marketing-agent.md          # Marketing sub-agent scope & capabilities
├── email-agent.md              # Email marketing & communication sub-agent
├── support-agent.md            # Support sub-agent (chat widget, WhatsApp, email support)
├── self-healing-agent.md       # Self-healing sub-agent scope
└── integration.md              # Technical integration, data flow, error handling
```

---

**See Also:**
- [Master Agent Architecture](./master-agent.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Marketing Agent](./marketing-agent.md)
- [Email Agent](./email-agent.md)
- [Support Agent](./support-agent.md)
- [Self-Healing Agent](./self-healing-agent.md)
- [Technical Integration & Workflow](./integration.md)
- [Architecture Overview](../overview.md)
- [WordPress Architecture](../wordpress/README.md)
- [Laravel Architecture](../laravel/README.md)
