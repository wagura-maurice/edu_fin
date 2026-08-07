# Technical Integration & Workflow

## Data Flow, Communication Protocols, Error Handling & Escalation

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Integration Overview](#1-integration-overview)
2. [Communication Protocols](#2-communication-protocols)
3. [Data Flow Architecture](#3-data-flow-architecture)
4. [WordPress Integration Points](#4-wordpress-integration-points)
5. [Laravel Integration Points](#5-laravel-integration-points)
6. [External Service Integration](#6-external-service-integration)
7. [Error Handling & Escalation](#7-error-handling--escalation)
8. [Security Boundaries](#8-security-boundaries)
9. [Observability & Monitoring](#9-observability--monitoring)

---

## 1. Integration Overview

The AI Agents module integrates with the existing EduFin platform through well-defined interfaces. The integration is **non-invasive**: agents communicate with WordPress and Laravel through their existing APIs, SSH access, and standard protocols. No modifications to core WordPress or Laravel business logic are required.

### Integration Topology

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                        AI AGENTS INTEGRATION TOPOLOGY                                │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│                         ┌─────────────────┐                                       │
│                         │  MASTER AGENT   │                                       │
│                         │  (Orchestrator) │                                       │
│                         └──┬───┬──┬───┬──────┘                                    │
│                            │   │  │   │                                            │
│               ┌────────────┘   │  │   └────────────┐                             │
│               │                │  │                │                              │
│               ▼                ▼  ▼                ▼                              │
│      ┌────────────┐   ┌────────────┐   ┌──────────┐  ┌──────────────┐            │
│      │ MARKETING  │   │   EMAIL    │   │ SUPPORT  │  │ SELF-HEALING │            │
│      │   AGENT    │   │   AGENT    │   │  AGENT   │  │    AGENT     │            │
│      └──┬───┬────┘   └─────┬──────┘   └──┬───┬───┘  └──┬───┬───────┘            │
│         │   │              │              │   │       │   │                      │
│         │   │              │              │   │       │   │                      │
│    ┌────┘   └────┐    ┌────┘         ┌────┘   └───┐   │   └────┐                │
│    │             │    │              │            │   │        │                 │
│    ▼             ▼    ▼              ▼            ▼   ▼        ▼                 │
│  ┌──────┐    ┌──────┐ ┌──────┐  ┌──────┐  ┌──────┐ ┌──────┐ ┌──────────┐       │
│  │Social│    │ WP   │ │ SMTP │  │ WP   │  │ WAHA │ │ SMTP │ │ Git Repo│       │
│  │Media │    │ REST │ │ IMAP │  │ Chat │  │ WA   │ │ IMAP │ │ SSH     │       │
│  │ APIs │    │ API  │ │      │  │Widget│  │      │ │      │ │ Servers │       │
│  └──────┘    └──┬───┘ └──────┘  └──────┘  └──────┘ └──────┘ └─────┬────┘       │
│                 │                                  │                              │
│                 │          ┌──────────────┐       │                              │
│                 └─────────►│   LARAVEL    │◄──────┘                              │
│                            │   REST API   │                                      │
│                            │ edufin.co.ke │                                      │
│                            │  /api/v1     │                                      │
│                            └──────┬───────┘                                      │
│                                   │                                               │
│                                   ▼                                               │
│                            ┌──────────────┐                                      │
│                            │ POSTGRESQL   │                                      │
│                            │ (Audit Log   │                                      │
│                            │  Storage)    │                                      │
│                            └──────────────┘                                      │
│                                                                                     │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                              EXISTING EDUFIN PLATFORM                               │
│                                                                                     │
│   ┌─────────────────────┐              ┌─────────────────────────────────┐        │
│   │   WORDPRESS         │              │   LARAVEL                       │        │
│   │   edufin.co.ke      │              │   app.edufin.co.ke              │        │
│   │   MySQL 8.0         │              │   PostgreSQL 16                 │        │
│   └─────────────────────┘              └─────────────────────────────────┘        │
│                                                                                     │
│   ┌─────────────────────────────────────────────────────────────────┐             │
│   │   CLOUDFLARE (DNS, CDN, WAF, SSL)                              │             │
│   └─────────────────────────────────────────────────────────────────┘             │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Communication Protocols

### 2.1 Protocol Summary

| Communication Path | Protocol | Transport | Format |
|---------------------|----------|-----------|--------|
| Master Agent ↔ Sub-Agent | MCP (JSON-RPC 2.0) | stdio / SSE | JSON |
| Marketing Agent ↔ Social Media APIs | X/Twitter, Facebook Graph, Instagram Graph, TikTok Business, LinkedIn Marketing, YouTube Data APIs | HTTPS | JSON |
| Marketing Agent ↔ WordPress | WordPress REST API | HTTPS | JSON |
| Marketing Agent ↔ SMTP (marketing@) | SMTP / IMAP | TLS / SSL | RFC 5321/3501 |
| Marketing Agent ↔ WAHA | WAHA REST API | HTTPS | JSON |
| Email Agent ↔ SMTP/IMAP | SMTP / IMAP | TLS / SSL | RFC 5321/3501 |
| Support Agent ↔ WordPress Chat Widget | WebSocket | WSS | JSON |
| Support Agent ↔ WAHA (WhatsApp) | WAHA REST API | HTTPS | JSON |
| Support Agent ↔ SMTP/IMAP | SMTP / IMAP | TLS / SSL | RFC 5321/3501 |
| Self-Healing Agent ↔ Git | Git over SSH | SSH | Git protocol |
| Self-Healing Agent ↔ Servers | SSH | SSH (key auth) | Shell commands |
| All Agents ↔ Laravel API | REST API | HTTPS | JSON |
| All Agents ↔ Redis | Redis Protocol | TCP | RESP |
| Master Agent ↔ HITL Interface | WebSocket + REST | HTTPS/WSS | JSON |

### 2.2 Master Agent ↔ Sub-Agent Communication

All Master Agent to sub-agent communication follows the MCP protocol (see [MCP Protocol Specification](./mcp-protocol.md) for full details):

```
COMMUNICATION SEQUENCE (typical task)
─────────────────────────────────────

Master Agent                    Sub-Agent
     │                              │
     │  1. initialize ──────────►  │
     │  ◄──── serverInfo ─────────  │
     │                              │
     │  2. tools/list ──────────►  │
     │  ◄──── tool schemas ──────  │
     │                              │
     │  3. tools/call ──────────►  │
     │     (post_tweet)            │
     │                              │
     │  ◄── notifications/progress │  (optional, for long tasks)
     │                              │
     │  ◄──── result ─────────────  │
     │                              │
     │  4. (task complete)          │
     │                              │
```

### 2.3 Sub-Agent ↔ External Service Communication

Sub-agents communicate with external services using their native protocols. The Master Agent does not participate in these connections — it only routes tasks and receives results.

| Sub-Agent | External Service | Protocol | Connection Owner |
|-----------|-----------------|----------|-----------------|
| Marketing | Social Media APIs (X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube) | HTTPS REST | Marketing Agent (direct) |
| Marketing | WordPress REST | HTTPS REST | Marketing Agent (direct) |
| Marketing | SMTP (marketing@edufin.co.ke) | SMTP/TLS | Marketing Agent (direct) |
| Marketing | WAHA (WhatsApp) | HTTPS REST | Marketing Agent (direct) |
| Email | SMTP server (all mailboxes) | SMTP/TLS | Email Agent (direct) |
| Email | IMAP server (all mailboxes) | IMAP/SSL | Email Agent (direct) |
| Support | WordPress Chat Widget | WebSocket (WSS) | Support Agent (direct) |
| Support | WAHA (WhatsApp) | HTTPS REST | Support Agent (direct) |
| Support | SMTP/IMAP (support@, customer_care@) | SMTP/TLS, IMAP/SSL | Support Agent (direct) |
| Self-Healing | Git repository | SSH | Self-Healing Agent (direct) |
| Self-Healing | WordPress server | SSH | Self-Healing Agent (direct) |
| Self-Healing | Laravel server | SSH | Self-Healing Agent (direct) |

---

## 3. Data Flow Architecture

### 3.1 Data Flow: Marketing Campaign

```
MARKETING CAMPAIGN DATA FLOW
────────────────────────────

1. TRIGGER: Scheduled (Monday 09:00 EAT)
   │
   ▼
2. MASTER AGENT: Create task "weekly_marketing_campaign"
   │
   ▼
3. MASTER AGENT → MARKETING AGENT: tools/call analyze_trends()
   │
   ├── Marketing Agent → X/Twitter API: GET /2/trends (Kenya)
   ├── Marketing Agent → X/Twitter API: GET /2/tweets/search (keywords)
   │
   ▼
4. MARKETING AGENT → MASTER AGENT: Return trend analysis
   │
   ▼
5. MASTER AGENT → MARKETING AGENT: tools/call get_product_info()
   │
   ├── Marketing Agent → Laravel API: GET /api/v1/packages
   │
   ▼
6. MARKETING AGENT → MASTER AGENT: Return product data
   │
   ▼
7. MASTER AGENT → MARKETING AGENT: tools/call generate_content()
   │
   ├── Marketing Agent → LLM: Generate tweet variants
   │
   ▼
8. MARKETING AGENT → MASTER AGENT: Return content drafts
   │
   ▼
9. MASTER AGENT: HITL EVALUATION
   │
   ├── Content contains promotional claims → HITL required
   │
   ▼
10. MASTER AGENT → MANAGEMENT: HITL approval request (Slack + Dashboard)
    │
    ├── Management reviews and approves
    │
    ▼
11. MASTER AGENT → MARKETING AGENT: tools/call post_tweet()
    │
    ├── Marketing Agent → X/Twitter API: POST /2/tweets
    │
    ▼
12. MARKETING AGENT → MASTER AGENT: Return tweet ID + timestamp
    │
    ▼
13. MASTER AGENT: Log task completion to Redis + PostgreSQL (via Laravel API)
    │
    ▼
14. COMPLETE
```

### 3.2 Data Flow: Email Auto-Response

```
EMAIL AUTO-RESPONSE DATA FLOW
─────────────────────────────

1. TRIGGER: IMAP poll detects new email in info@edufin.co.ke
   │
   ▼
2. EMAIL AGENT: Fetch and parse email
   │
   ▼
3. EMAIL AGENT → MASTER AGENT: tools/call categorize_email()
   │  (via notifications — agent reports new inbound)
   │
   ▼
4. MASTER AGENT → EMAIL AGENT: tools/call categorize_email()
   │
   ├── Email Agent → LLM: Classify email content
   │
   ▼
5. EMAIL AGENT → MASTER AGENT: Return category (e.g., "faq_general")
   │
   ▼
6. MASTER AGENT: Evaluate auto-response eligibility
   │
   ├── Category = faq_general → Auto-respond with template
   │
   ▼
7. MASTER AGENT → EMAIL AGENT: tools/call send_response()
   │  response_type = "auto_template"
   │
   ├── Email Agent → SMTP: Send response
   ├── Email Agent → IMAP: Move original to Agent_Processed
   │
   ▼
8. EMAIL AGENT → MASTER AGENT: Return send confirmation
   │
   ▼
9. MASTER AGENT: Log to Redis + PostgreSQL (audit trail)
   │
   ▼
10. COMPLETE
```

### 3.3 Data Flow: Self-Healing Repair

```
SELF-HEALING REPAIR DATA FLOW
─────────────────────────────

1. TRIGGER: Health check detects Laravel Horizon workers down
   │
   ▼
2. SELF-HEALING AGENT → MASTER AGENT: Report anomaly
   │
   ▼
3. MASTER AGENT → SELF-HEALING AGENT: tools/call run_diagnostic()
   │
   ├── Self-Healing Agent → SSH (Laravel server): Read Horizon logs
   ├── Self-Healing Agent → SSH (Laravel server): Check process list
   │
   ▼
4. SELF-HEALING AGENT → MASTER AGENT: Return diagnostic report
   │  (Root cause: OOM kill, memory limit too low)
   │
   ▼
5. MASTER AGENT: Evaluate repair strategy
   │
   ├── Issue = service restart → HITL Level 1
   │
   ▼
6. MASTER AGENT → MANAGEMENT: HITL request (restart_service)
   │
   ├── Management approves
   │
   ▼
7. MASTER AGENT → SELF-HEALING AGENT: tools/call restart_service()
   │  service = "horizon"
   │
   ├── Self-Healing Agent → SSH (Laravel server): systemctl restart horizon
   ├── Self-Healing Agent → SSH (Laravel server): Verify workers running
   │
   ▼
8. SELF-HEALING AGENT → MASTER AGENT: Return success + health verification
   │
   ▼
9. MASTER AGENT: Monitor service for 30 minutes
   │
   ├── Service stable → Mark issue RESOLVED
   └── Service fails again → Escalate (Level 2)
   │
   ▼
10. MASTER AGENT: Log to Redis + PostgreSQL (audit trail)
    │
    ▼
11. COMPLETE
```

---

## 4. WordPress Integration Points

### 4.1 Integration Methods

| Method | Used By | Purpose | Auth |
|--------|---------|---------|------|
| WordPress REST API | Marketing Agent | Create blog drafts, list posts, upload media, SEO suggestions | Application Password |
| WordPress Chat Widget (WebSocket) | Support Agent | Real-time chat with website visitors | API Key + WebSocket token |
| SSH access | Self-Healing Agent | Log access, service restarts, health checks | SSH key |
| Webhook (outbound) | WordPress → Master Agent | Notify on new inquiry, newsletter signup, chat escalation | HMAC signature |

### 4.2 WordPress REST API Endpoints Used

| Endpoint | Method | Agent | Purpose |
|----------|--------|-------|---------|
| `/wp-json/wp/v2/posts` | GET | Marketing | List existing posts (avoid duplication) |
| `/wp-json/wp/v2/posts` | POST | Marketing | Create draft blog post |
| `/wp-json/wp/v2/media` | POST | Marketing | Upload featured image for blog post |
| `/wp-json/wp/v2/pages` | GET | Marketing | Read page content for reference |
| `/wp-json/wp/v2/users/me` | GET | Marketing | Verify authentication |

### 4.3 WordPress Webhook Integration

WordPress sends webhooks to the Master Agent for event-driven tasks:

| WordPress Event | Webhook Target | Triggered Task |
|-----------------|---------------|----------------|
| New contact form submission | `POST /agents/webhooks/wp/inquiry` | Email Agent: Send acknowledgment + draft response |
| New newsletter subscriber | `POST /agents/webhooks/wp/subscriber` | Email Agent: Send welcome email |
| New comment on blog post | `POST /agents/webhooks/wp/comment` | Marketing Agent: Analyze sentiment; escalate if negative |
| Chat widget escalation | `POST /agents/webhooks/wp/chat/escalate` | Support Agent: Escalate chat to human staff; notify via Slack |
| Chat widget offline message | `POST /agents/webhooks/wp/chat/offline` | Support Agent: Collect contact info; create follow-up task |

### 4.4 WordPress SSH Access (Self-Healing Agent)

| Operation | Command Example | Purpose |
|-----------|----------------|---------|
| Health check | `systemctl status php8.2-fpm` | Check PHP-FPM status |
| Log access | `tail -100 /var/log/nginx/error.log` | Read Nginx error logs |
| Service restart | `systemctl restart php8.2-fpm` | Restart PHP-FPM |
| Cache clear | `wp cache flush --path=/var/www/wordpress` | Clear WordPress cache |
| Disk check | `df -h` | Check disk space |
| Process check | `ps aux \| grep php` | Check PHP processes |

> **Security:** SSH access is restricted to a dedicated user account (`edufin-agent`) with limited sudo permissions for service restarts only. The agent cannot access MySQL credentials, WordPress config secrets, or user data via SSH.

---

## 5. Laravel Integration Points

### 5.1 Integration Methods

| Method | Used By | Purpose | Auth |
|--------|---------|---------|------|
| REST API (read-only) | Marketing Agent, Email Agent, Support Agent | Fetch product info, subscriber lists, FAQ content, notification triggers | API Key (`X-API-Key`) |
| REST API (webhook receiver) | Master Agent | Receive webhooks from Laravel events | HMAC signature |
| SSH access | Self-Healing Agent | Log access, service restarts, health checks, artisan commands | SSH key |
| PostgreSQL (indirect) | Master Agent | Audit log storage (via Laravel API endpoint) | API Key |

### 5.2 Laravel REST API Endpoints Used

| Endpoint | Method | Agent | Purpose |
|----------|--------|-------|---------|
| `GET /api/v1/packages` | GET | Marketing, Support | Fetch financing packages for content generation / FAQ answers |
| `GET /api/v1/packages/{slug}` | GET | Marketing, Support | Fetch specific package details |
| `GET /api/v1/calculator/rates` | GET | Marketing | Fetch current rates for content accuracy |
| `GET /api/v1/faqs` | GET | Support | Fetch FAQ content for chat widget / WhatsApp / email responses |
| `GET /api/v1/health` | GET | Self-Healing | Check Laravel application health |
| `POST /api/v1/agents/audit` | POST | Master Agent | Submit agent action audit log |
| `POST /api/v1/agents/hitl` | POST | Master Agent | Submit HITL request record |
| `POST /api/v1/agents/webhooks/lv/event` | POST | Laravel → Master Agent | Laravel event notifications |

### 5.3 Laravel Event Webhooks

Laravel sends webhooks to the Master Agent for business events that may trigger agent tasks:

| Laravel Event | Webhook Target | Triggered Task |
|---------------|---------------|----------------|
| New user registered | `POST /agents/webhooks/lv/registered` | Email Agent: Send welcome sequence |
| Loan application submitted | `POST /agents/webhooks/lv/loan_applied` | Email Agent: Send confirmation + next steps |
| Loan approved | `POST /agents/webhooks/lv/loan_approved` | Email Agent: Send approval notification |
| Payment received | `POST /agents/webhooks/lv/payment` | Email Agent: Send receipt (if opted in) |
| KYC verified | `POST /agents/webhooks/lv/kyc_verified` | Email Agent: Send verification confirmation |
| System error (critical) | `POST /agents/webhooks/lv/error` | Self-Healing Agent: Run diagnostic |

### 5.4 Laravel SSH Access (Self-Healing Agent)

| Operation | Command Example | Purpose |
|-----------|----------------|---------|
| Health check | `systemctl status php8.3-fpm` | Check PHP-FPM status |
| Horizon status | `php artisan horizon:status` | Check queue worker status |
| Log access | `tail -100 /var/www/laravel/storage/logs/laravel.log` | Read Laravel error logs |
| Cache clear | `php artisan cache:clear` | Clear application cache |
| Service restart | `systemctl restart horizon` | Restart Horizon queue workers |
| Config cache | `php artisan config:cache` | Rebuild config cache |
| Failed jobs | `php artisan queue:failed` | List failed jobs |
| Disk check | `df -h` | Check disk space |

> **Security:** SSH access uses a dedicated user account (`edufin-agent`) with limited sudo permissions. The agent cannot run `php artisan tinker`, `php artisan db:seed`, `php artisan migrate`, or any command that modifies the database schema or accesses raw data. These commands are blocked via sudoers configuration.

### 5.5 Audit Log Integration

Agent actions are persisted to the Laravel PostgreSQL database for long-term audit storage via a dedicated API endpoint:

**Endpoint:** `POST /api/v1/agents/audit`

```json
{
  "agent": "marketing-agent",
  "task_id": "task-uuid-1234",
  "action": "post_tweet",
  "timestamp": "2026-08-08T14:00:02Z",
  "details": {
    "tool": "post_tweet",
    "arguments": { "content": "..." },
    "result": { "tweet_id": "1823..." }
  },
  "hitl_approved": true,
  "hitl_approved_by": "ops.manager@edufin.co.ke",
  "correlation_id": "corr-abc-123"
}
```

> **Security:** The audit endpoint accepts writes only (no reads). It is authenticated via API key and rate-limited. Audit records are immutable once written.

---

## 6. External Service Integration

### 6.1 Social Media API Integrations

The Marketing Agent integrates with multiple social media platforms used in Kenya:

| Platform | API | Account | Auth Method | Used By |
|----------|-----|---------|-------------|---------|
| X/Twitter | X API v2 | `@EduFinKe` | OAuth 2.0 (User Context) + Bearer Token | Marketing Agent |
| Facebook | Facebook Graph API | EduFin Kenya page | Page Access Token | Marketing Agent |
| Instagram | Instagram Graph API | `edufin.ke` | Instagram Basic Display API + Graph API token | Marketing Agent |
| TikTok | TikTok Business API | `edufin.ke` | OAuth 2.0 | Marketing Agent |
| LinkedIn | LinkedIn Marketing API | EduFin company page | OAuth 2.0 | Marketing Agent |
| YouTube | YouTube Data API v3 | EduFin channel | OAuth 2.0 | Marketing Agent |

**Common Integration Aspects:**

| Aspect | Specification |
|--------|---------------|
| Rate Limit Handling | Respect platform-specific rate limit headers |
| Retry Strategy | Exponential backoff on 429 (rate limit) and 5xx errors |
| Webhook | Platform-specific webhooks for mention/reply/comment notifications |
| Content Format | Platform-appropriate (text, images, video, carousel) |

### 6.2 SMTP/IMAP Integration

| Aspect | Specification |
|--------|---------------|
| SMTP Host | `mail.edufin.co.ke:587` (STARTTLS) |
| IMAP Host | `mail.edufin.co.ke:993` (SSL) |
| Auth | SMTP Auth / IMAP Auth (per mailbox) |
| Mailboxes | `info@`, `support@`, `marketing@`, `customer_care@`, `communications@edufin.co.ke` |
| Polling | IMAP IDLE or 2-minute polling interval |
| Sending Limits | 100/hour, 500/day per mailbox |
| Used By | Email Agent (all mailboxes), Marketing Agent (marketing@), Support Agent (support@, customer_care@) |

### 6.3 WAHA (WhatsApp) Integration

| Aspect | Specification |
|--------|---------------|
| WAHA Server URL | `http://waha.edufin.co.ke` (internal) |
| API Version | WAHA REST API |
| Auth | API Key (header: `X-API-Key`) |
| WhatsApp Business Number | Configured in WAHA server |
| Send Message | `POST /api/sendText` (text), `POST /api/sendImage` (image) |
| Receive Message | WAHA webhook → `POST /agents/webhooks/whatsapp/inbound` |
| Delivery Report | WAHA webhook → `POST /agents/webhooks/whatsapp/delivery` |
| Session Management | WAHA maintains WhatsApp session; agent monitors connection status |
| Used By | Marketing Agent (broadcasts to opted-in subscribers), Support Agent (customer support conversations) |
| Rate Limit | Max 50 messages/hour (configurable); respect WhatsApp anti-spam policies |

### 6.4 WordPress Chat Widget Integration

| Aspect | Specification |
|--------|---------------|
| Widget Location | Bottom-right corner of all WordPress pages |
| Communication | WebSocket (WSS) connection to Support Agent backend |
| Auth | API Key + per-session WebSocket token |
| Rate Limit | Max 10 messages/minute per visitor |
| Used By | Support Agent |
| Offline Behavior | Displays contact form; collects visitor info for follow-up |

### 6.5 Git Integration

| Aspect | Specification |
|--------|---------------|
| Repository | EduFin monorepo (WordPress + Laravel) |
| Access | SSH deploy key (read + branch creation) |
| Branch Naming | `agent/fix-{issue_id}`, `agent/feature-{task_id}` |
| Commit Format | Conventional Commits with agent attribution |
| PR Creation | Via `gh` CLI (GitHub) or GitLab API |
| Merge | Never autonomous — PRs require human review + HITL approval |

### 6.6 Cloudflare Integration

| Aspect | Specification |
|--------|---------------|
| API | Cloudflare REST API v4 |
| Auth | API Token (scoped: read-only + cache purge) |
| Used By | Self-Healing Agent |
| Capabilities | Read zone status, purge cache, check SSL status |
| Restrictions | Cannot modify DNS, WAF rules, or access policies |

---

## 7. Error Handling & Escalation

### 7.1 Error Classification

| Error Type | Examples | Initial Handling | Escalation |
|------------|----------|------------------|------------|
| **Transient** | Network timeout, rate limit, temporary 5xx | Auto-retry with backoff (max 3) | None if resolved |
| **Tool Error** | Invalid arguments, schema validation failure | Log error, return failure to Master Agent | Master Agent decides: retry or escalate |
| **Agent Error** | Sub-agent crash, MCP connection lost | Master Agent circuit breaker; attempt reconnect | If unrecoverable: Level 2 escalation |
| **External Service Error** | X API down, SMTP server unreachable | Retry with backoff; queue task | If persistent > 15 min: Level 1 notification |
| **Security Error** | Auth failure, permission denied, PII detected in output | Block action immediately | Level 2 escalation; security review |
| **Data Error** | Unexpected response format, missing fields | Log anomaly; return to Master Agent | Master Agent HITL review |
| **Infrastructure Error** | Server down, disk full, OOM | Self-Healing Agent diagnostic | Level 2 or Level 3 based on severity |

### 7.2 Error Handling Flow

```
ERROR HANDLING FLOW
───────────────────

1. ERROR OCCURS (in sub-agent tool execution)
   │
   ▼
2. SUB-AGENT: Classify error
   │
   ├── Transient ──► Retry (backoff: 1s, 5s, 15s)
   │                   │
   │                   ├── Retry succeeds ──► Return result
   │                   └── Retry exhausted ──► Return error to Master Agent
   │
   ├── Tool Error ──► Return error to Master Agent (no retry)
   │
   ├── Security Error ──► Block + return error + flag for review
   │
   └── Other ──► Return error to Master Agent
   │
   ▼
3. MASTER AGENT: Receive error
   │
   ├── Retryable ──► Re-dispatch task (max 2 re-dispatches)
   │                   │
   │                   ├── Success ──► Continue
   │                   └── Failure ──► Escalate
   │
   ├── Non-retryable ──► Evaluate escalation level
   │
   └── Security ──► Immediate Level 2 escalation
   │
   ▼
4. ESCALATION (per level)
   │
   ├── Level 1 (HITL) ──► Pause task; request management approval
   ├── Level 2 (Review) ──► Notify management; task marked FAILED
   └── Level 3 (Critical) ──► Urgent notification; pause all non-critical tasks
   │
   ▼
5. LOG & AUDIT
   │  All errors, retries, and escalations logged to audit trail
```

### 7.3 Escalation Matrix

| Scenario | Escalation Level | Notification Channel | Timeout |
|----------|-----------------|----------------------|---------|
| Tool requires HITL approval | Level 1 | Slack + Dashboard | 24 hours |
| Sub-agent fails after retries | Level 1 | Slack + Dashboard | 4 hours |
| External service down > 15 min | Level 1 | Slack + Dashboard | 4 hours |
| Self-Healing can't fix issue | Level 2 | Slack (urgent) + Email | 4 hours |
| Security error detected | Level 2 | Slack (urgent) + Email | 2 hours |
| Agent confidence < 0.70 | Level 2 | Slack + Dashboard | 4 hours |
| Critical service down | Level 3 | Slack (@here) + WhatsApp + Phone | 30 minutes |
| Multiple services failing | Level 3 | Slack (@here) + WhatsApp + Phone | 30 minutes |
| HITL approval timeout (24h) | Level 2 | Slack (urgent) + Email | 4 hours |

### 7.4 Sub-Agent Failure Recovery

When a sub-agent becomes unresponsive or fails repeatedly, the Master Agent follows this recovery procedure:

| Step | Action | Condition |
|------|--------|-----------|
| 1 | Mark agent as `degraded` | First failure detected |
| 2 | Attempt MCP reconnection | Connection lost |
| 3 | Queue tasks for the agent | Agent unavailable but recoverable |
| 4 | Mark agent as `unhealthy` | > 3 consecutive failures |
| 5 | Open circuit breaker | > 5 consecutive failures |
| 6 | Escalate to management (Level 2) | Circuit breaker open |
| 7 | Attempt agent process restart | If configured and permitted |
| 8 | Mark agent as `offline` | Restart failed or not configured |
| 9 | Escalate to Level 3 | Agent offline and tasks are critical |

---

## 8. Security Boundaries

### 8.1 Agent Permission Matrix

| Capability | Master Agent | Marketing Agent | Email Agent | Support Agent | Self-Healing Agent |
|------------|:------------:|:---------------:|:-----------:|:-------------:|:------------------:|
| Route tasks to sub-agents | Yes | - | - | - | - |
| Enforce HITL | Yes | - | - | - | - |
| Post to social media (all platforms) | - | Yes | - | - | - |
| Create WordPress drafts | - | Yes | - | - | - |
| Suggest WordPress SEO optimizations | - | Yes | - | - | - |
| Send marketing emails (marketing@) | - | Yes | - | - | - |
| Send WhatsApp broadcasts (WAHA) | - | Yes | - | - | - |
| Send emails (SMTP, all mailboxes) | - | - | Yes | - | - |
| Read emails (IMAP, all mailboxes) | - | - | Yes | - | - |
| Power WordPress chat widget | - | - | - | Yes | - |
| Send WhatsApp support messages (WAHA) | - | - | - | Yes | - |
| Send support emails (support@, customer_care@) | - | - | - | Yes | - |
| Escalate conversations to staff | - | - | - | Yes | - |
| SSH to servers | - | - | - | - | Yes |
| Git operations | - | - | - | - | Yes |
| Read Laravel API (public) | - | Yes | Yes | Yes | Yes |
| Write Laravel audit API | Yes | - | - | - | - |
| Access PII/financial data | No | No | No | No | No |
| Modify database schema | No | No | No | No | No |
| Modify security config | No | No | No | No | No |

### 8.2 Data Access Restrictions

| Data Type | Access | Rationale |
|-----------|--------|-----------|
| WordPress content (public) | Read (Marketing Agent, Support Agent) | For content reference, blog creation, SEO suggestions, FAQ answers |
| WordPress user data | No access | PII; not needed for agent functions |
| Laravel public API data (packages, rates, FAQs) | Read (all agents) | For content accuracy, FAQ responses, and health checks |
| Laravel user/client data | No access | PII; agents do not need client data |
| Laravel financial data (loans, payments) | No access | Financial data; agents do not need transaction data |
| Email content (inbound/outbound) | Read/write (Email Agent, Support Agent for support mailboxes, Marketing Agent for marketing@) | Required for email management and support |
| Chat widget conversations | Read/write (Support Agent only) | Required for real-time chat support |
| WhatsApp conversations | Read/write (Support Agent for support, Marketing Agent for broadcasts) | Required for WhatsApp support and marketing |
| Server logs | Read (Self-Healing Agent) | Required for diagnostics |
| Git source code | Read + branch write (Self-Healing Agent) | Required for patch generation |
| Agent audit logs | Write (Master Agent) | For compliance and traceability |

### 8.3 Network Security

| Boundary | Protection |
|----------|-----------|
| Agent ↔ Internet | Cloudflare WAF, rate limiting (inherited from existing platform) |
| Agent ↔ WordPress server | HTTPS (REST API), SSH (key auth, restricted user) |
| Agent ↔ WordPress chat widget | WSS (WebSocket Secure) with per-session token |
| Agent ↔ Laravel server | HTTPS (REST API), SSH (key auth, restricted user) |
| Agent ↔ SMTP/IMAP | TLS/SSL encryption |
| Agent ↔ WAHA (WhatsApp) | HTTPS with API key (internal network) |
| Master ↔ Sub-agents (local) | stdio (no network exposure) |
| Master ↔ Sub-agents (remote) | TLS 1.2+ over SSE |
| Agent ↔ Redis | TCP with AUTH password |
| Agent ↔ LLM API | HTTPS with API key |

---

## 9. Observability & Monitoring

### 9.1 Agent Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Tasks completed | Master Agent | Throughput measurement |
| Tasks failed | Master Agent | Error rate monitoring |
| HITL requests pending | Master Agent | Approval backlog |
| HITL approval time | Master Agent | Management responsiveness |
| Tool call latency | All agents | Performance monitoring |
| Tool call success rate | All agents | Reliability monitoring |
| Agent uptime | Master Agent health checks | Availability tracking |
| Escalation count | Master Agent | Issue frequency tracking |
| External API calls | Sub-agents | Rate limit monitoring |

### 9.2 Audit Trail

Every agent action is recorded in an immutable audit trail:

| Audit Record Field | Description |
|--------------------|-------------|
| `timestamp` | When the action occurred |
| `agent` | Which agent performed the action |
| `task_id` | Correlation to the originating task |
| `tool` | Which MCP tool was called |
| `arguments` | Input arguments (sanitized — no secrets) |
| `result` | Tool output (sanitized — no PII) |
| `hitl_required` | Whether HITL was triggered |
| `hitl_approved_by` | Who approved (if HITL) |
| `error` | Error details (if failed) |
| `correlation_id` | End-to-end trace ID |

### 9.3 Dashboard

The HITL dashboard provides management with real-time visibility into agent activity:

| Dashboard Section | Content |
|-------------------|---------|
| **Active Tasks** | Currently executing tasks with progress |
| **HITL Queue** | Pending approval requests |
| **Recent Actions** | Last 50 agent actions with details |
| **Agent Health** | Status of each sub-agent (healthy/degraded/unhealthy/offline) |
| **Escalations** | Open escalations with severity levels |
| **Metrics** | Daily/weekly metrics charts |
| **Audit Log** | Searchable audit trail |

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Master Agent Architecture](./master-agent.md)
- [Marketing Agent](./marketing-agent.md)
- [Email Agent](./email-agent.md)
- [Support Agent](./support-agent.md)
- [Self-Healing Agent](./self-healing-agent.md)
- [Architecture Overview](../overview.md)
- [Security Documentation](../../security/README.md)
- [Deployment Documentation](../../deployment/README.md)
