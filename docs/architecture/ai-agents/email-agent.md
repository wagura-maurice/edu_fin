# Email Agent

## Email Marketing & Communication Sub-Agent

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Agent Overview](#1-agent-overview)
2. [MCP Server Definition](#2-mcp-server-definition)
3. [SMTP Integration Requirements](#3-smtp-integration-requirements)
4. [Conversation Initiation](#4-conversation-initiation)
5. [Thread Tracking & State Management](#5-thread-tracking--state-management)
6. [Automated Response Logic](#6-automated-response-logic)
7. [HITL Triggers](#7-hitl-triggers)
8. [Scheduled Workflows](#8-scheduled-workflows)

---

## 1. Agent Overview

The Email Agent is a specialized sub-agent responsible for **email communication management** across EduFin's primary mailboxes. It handles conversation initiation, thread tracking, automated responses to common inquiries, and email marketing campaigns. It operates as an MCP Server exposing email-related tools to the Master Agent.

### Scope Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        EMAIL AGENT - SCOPE                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  IN SCOPE:                                                                      │
│  ✓ Monitor inbound emails on info@edufin.co.ke and support@edufin.co.ke       │
│  ✓ Initiate outbound email conversations (campaigns, follow-ups)               │
│  ✓ Track email threads and conversation state                                  │
│  ✓ Generate and send automated responses to common inquiries                   │
│  ✓ Categorize and prioritize inbound emails                                    │
│  ✓ Send scheduled email campaigns to subscriber lists                          │
│  ✓ Detect when an email requires human (staff) response                        │
│  ✓ Draft responses for staff review and approval                               │
│                                                                                 │
│  OUT OF SCOPE:                                                                  │
│  ✗ Accessing Laravel user accounts or loan data directly                       │
│  ✗ Sending emails that contain PII without encryption                          │
│  ✗ Modifying DNS, SPF, DKIM, or DMARC records                                 │
│  ✗ Handling emails marked as legal notices (escalate to HITL)                  │
│  ✗ Sending unsolicited marketing emails (spam) to non-subscribers              │
│  ✗ Auto-responding to emails containing financial transaction details          │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Managed Mailboxes

| Mailbox | Purpose | Agent Role |
|---------|---------|------------|
| `info@edufin.co.ke` | General inquiries, product questions, partnership requests | Monitor, auto-respond to FAQs, draft replies for staff |
| `support@edufin.co.ke` | Customer support, account issues, complaint handling | Monitor, categorize, draft replies, escalate urgent issues |

### External Service Dependencies

| Service | Purpose | Auth Method |
|---------|---------|-------------|
| SMTP Server (outbound) | Send emails | SMTP Auth (username + password) |
| IMAP Server (inbound) | Read incoming emails | IMAP Auth (username + password) |
| Laravel REST API | Fetch subscriber lists, notification triggers | API Key (`X-API-Key`) |
| WordPress REST API | Fetch newsletter subscriber list | Application Password |

---

## 2. MCP Server Definition

### 2.1 Server Metadata

```json
{
  "serverInfo": {
    "name": "edufin-email-agent",
    "version": "1.0.0",
    "description": "Email marketing and communication sub-agent for SMTP integration, thread tracking, and automated responses",
    "capabilities": {
      "tools": true,
      "resources": true,
      "prompts": true
    }
  }
}
```

### 2.2 Exposed Tools

| Tool Name | Purpose | HITL Required | Destructive |
|-----------|---------|---------------|-------------|
| `check_inbox` | Fetch and categorize new inbound emails | Never | No |
| `categorize_email` | Classify an email by type and priority | Never | No |
| `draft_response` | Generate a draft response for an inbound email | Never | No |
| `send_response` | Send an email response (auto or draft-approved) | Conditional | No |
| `initiate_conversation` | Start a new outbound email thread | Conditional | No |
| `send_campaign` | Send an email campaign to a subscriber list | Always | No |
| `follow_up` | Send a follow-up message in an existing thread | Conditional | No |
| `get_thread` | Retrieve the full history of an email thread | Never | No |
| `escalate_to_staff` | Flag an email for urgent staff attention | Never | No |

### 2.3 Exposed Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://email/mailbox/{address}/inbox` | JSON | Inbox for a specific mailbox |
| `edufin://email/threads/{thread_id}` | JSON | Full email thread history |
| `edufin://email/threads/active` | JSON | All active (unresolved) email threads |
| `edufin://email/campaigns/scheduled` | JSON | Scheduled email campaigns |
| `edufin://email/campaigns/sent` | JSON | Log of sent campaigns with metrics |
| `edufin://email/categories` | JSON | Email category definitions and rules |

### 2.4 Exposed Prompts

| Prompt Name | Arguments | Purpose |
|-------------|-----------|---------|
| `email_draft_response` | `thread_id`, `tone` | Draft a response to an email inquiry |
| `email_follow_up` | `thread_id`, `purpose` | Generate a follow-up message |
| `email_campaign` | `subject`, `audience`, `product` | Generate campaign email content |

---

## 3. SMTP Integration Requirements

### 3.1 SMTP Configuration

The Email Agent connects to EduFin's SMTP server for outbound email delivery. Configuration is stored in secure environment variables:

| Parameter | Value (Example) | Notes |
|-----------|-----------------|-------|
| SMTP Host | `mail.edufin.co.ke` | EduFin mail server |
| SMTP Port | `587` | STARTTLS |
| Encryption | `TLS` | Transport-level encryption |
| Auth Method | `LOGIN` | Standard SMTP auth |
| info@ Credentials | `info@edufin.co.ke` / `<password>` | Stored in Vault/env |
| support@ Credentials | `support@edufin.co.ke` / `<password>` | Stored in Vault/env |
| From Address | `info@edufin.co.ke` or `support@edufin.co.ke` | Per mailbox context |
| Reply-To | Same as From | Responses route back to monitored mailbox |

### 3.2 IMAP Configuration (Inbound)

| Parameter | Value | Notes |
|-----------|-------|-------|
| IMAP Host | `mail.edufin.co.ke` | Same mail server |
| IMAP Port | `993` | SSL |
| Encryption | `SSL` | Encrypted inbound |
| Polling Interval | Every 2 minutes | Configurable |
| Folders Monitored | `INBOX` | Primary inbox per mailbox |
| Processed Folder | `Agent_Processed` | Emails moved here after processing |
| Escalated Folder | `Agent_Escalated` | Emails requiring staff attention |

### 3.3 Email Deliverability Requirements

Before the Email Agent goes live, the following DNS records must be verified:

| Record | Type | Purpose | Status Check |
|--------|------|---------|--------------|
| `edufin.co.ke` | TXT (SPF) | Authorize SMTP server to send | `v=spf1 include:_spf.google.com ~all` (verify mail server is included) |
| `default._domainkey.edufin.co.ke` | TXT (DKIM) | Email signing & integrity | Verify DKIM key is published |
| `_dmarc.edufin.co.ke` | TXT (DMARC) | Email authentication policy | `v=DMARC1; p=quarantine; rua=mailto:admin@edufin.co.ke` |

> **Pre-requisite:** The Email Agent must verify SPF, DKIM, and DMARC records before sending any outbound emails. If any record is missing or misconfigured, the agent will refuse to send and escalate to management.

### 3.4 Sending Limits

| Limit | Value | Rationale |
|-------|-------|-----------|
| Max emails per hour | 100 | Prevent triggering spam filters |
| Max emails per day | 500 | Stay within SMTP server limits |
| Max recipients per email | 50 | BCC for campaigns |
| Min delay between sends | 2 seconds | Avoid burst-sending patterns |
| Retry on failure | 3 attempts with exponential backoff | Handle transient SMTP errors |

---

## 4. Conversation Initiation

### 4.1 Outbound Email Types

The Email Agent can initiate outbound conversations in the following scenarios:

| Scenario | From Address | Trigger | HITL Required |
|----------|-------------|---------|---------------|
| **Welcome email** | `info@edufin.co.ke` | New subscriber signup (WordPress webhook) | No (template) |
| **Campaign email** | `info@edufin.co.ke` | Scheduled campaign or manual trigger | Yes |
| **Follow-up inquiry** | `support@edufin.co.ke` | Unresolved thread > 48 hours | Conditional |
| **Product announcement** | `info@edufin.co.ke` | New product launched (Laravel API event) | Yes |
| **Re-engagement** | `info@edufin.co.ke` | Inactive subscriber > 90 days | Yes |
| **Statement notification** | `support@edufin.co.ke` | Laravel notification event | No (template) |

### 4.2 Initiate Conversation Tool

**Tool: `initiate_conversation`**

```json
{
  "name": "initiate_conversation",
  "description": "Initiate a new outbound email conversation from an EduFin mailbox. Requires HITL approval for marketing campaigns and product announcements.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "from_address": {
        "type": "string",
        "enum": ["info@edufin.co.ke", "support@edufin.co.ke"],
        "description": "Sending mailbox"
      },
      "to_address": {
        "type": "string",
        "format": "email",
        "description": "Recipient email address"
      },
      "to_addresses": {
        "type": "array",
        "items": { "type": "string", "format": "email" },
        "description": "Multiple recipients (BCC for campaigns)"
      },
      "subject": {
        "type": "string",
        "description": "Email subject line"
      },
      "body_html": {
        "type": "string",
        "description": "HTML email body"
      },
      "body_text": {
        "type": "string",
        "description": "Plain text fallback body"
      },
      "conversation_type": {
        "type": "string",
        "enum": ["welcome", "campaign", "follow_up", "announcement", "re_engagement", "notification"],
        "description": "Type of conversation being initiated"
      },
      "thread_metadata": {
        "type": "object",
        "description": "Additional metadata to attach to the thread for tracking",
        "properties": {
          "campaign_id": { "type": "string" },
          "subscriber_id": { "type": "string" },
          "source": { "type": "string" }
        }
      }
    },
    "required": ["from_address", "subject", "body_html", "conversation_type"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "conversation_type is 'campaign', 'announcement', or 're_engagement'",
    "destructive": false,
    "idempotent": false,
    "category": "email_outbound",
    "timeout_ms": 30000,
    "retry_policy": "auto"
  }
}
```

---

## 5. Thread Tracking & State Management

### 5.1 Thread Data Model

Every email conversation is tracked as a thread with persistent state:

```json
{
  "thread_id": "thread-uuid-abcd",
  "from_address": "info@edufin.co.ke",
  "subject": "Welcome to EduFin",
  "category": "welcome",
  "priority": "normal",
  "status": "active",
  "participants": ["info@edufin.co.ke", "client@example.com"],
  "messages": [
    {
      "message_id": "msg-uuid-001",
      "direction": "outbound",
      "from": "info@edufin.co.ke",
      "to": "client@example.com",
      "subject": "Welcome to EduFin",
      "body_preview": "Welcome to EduFin! We're excited to...",
      "sent_at": "2026-08-08T10:00:00Z",
      "sent_by": "email-agent"
    },
    {
      "message_id": "msg-uuid-002",
      "direction": "inbound",
      "from": "client@example.com",
      "to": "info@edufin.co.ke",
      "subject": "Re: Welcome to EduFin",
      "body_preview": "Thank you. I'd like to know more about...",
      "received_at": "2026-08-08T11:30:00Z",
      "categorized_as": "product_inquiry",
      "auto_response_eligible": true
    }
  ],
  "created_at": "2026-08-08T10:00:00Z",
  "last_activity_at": "2026-08-08T11:30:00Z",
  "auto_response_count": 0,
  "staff_escalated": false,
  "metadata": {
    "campaign_id": null,
    "subscriber_id": "sub-123",
    "source": "wordpress_newsletter"
  }
}
```

### 5.2 Thread Status States

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      EMAIL THREAD STATE MACHINE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────┐                                                          │
│   │  ACTIVE  │  Thread has unresolved inquiries                        │
│   └────┬─────┘                                                          │
│        │                                                                │
│        ├── New inbound reply ──► Evaluate response strategy             │
│        │                                                                │
│        ▼                                                                │
│   ┌──────────────┐                                                      │
│   │ AUTO_RESPOND │  Agent generating/sending automated response        │
│   └──────┬───────┘                                                      │
│          │                                                              │
│          ├── Response sent, awaiting reply ──► ACTIVE                   │
│          │                                                              │
│          ├── Cannot auto-respond ──► STAFF_PENDING                     │
│          │                                                              │
│          └── HITL required ──► HITL_PENDING                            │
│                                                                         │
│   ┌──────────────┐                                                      │
│   │STAFF_PENDING │  Awaiting staff response (draft may be prepared)    │
│   └──────┬───────┘                                                      │
│          │                                                              │
│          ├── Staff responds ──► RESOLVED                               │
│          │                                                              │
│          └── No response > 48h ──► ESCALATED                           │
│                                                                         │
│   ┌──────────┐     ┌───────────┐                                       │
│   │ RESOLVED │     │ ESCALATED │  Urgent staff attention needed       │
│   └──────────┘     └───────────┘                                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Thread Correlation

Threads are correlated using standard email headers:

| Header | Purpose |
|--------|---------|
| `Message-ID` | Unique identifier per email message |
| `In-Reply-To` | Links a reply to the original message |
| `References` | Full thread chain of Message-IDs |
| `X-EduFin-Thread-ID` | Custom header injected by Email Agent for thread tracking |
| `X-EduFin-Category` | Custom header for email categorization |

---

## 6. Automated Response Logic

### 6.1 Email Categorization

Inbound emails are categorized to determine the appropriate response strategy:

| Category | Description | Auto-Response | Priority |
|----------|-------------|---------------|----------|
| `faq_general` | General questions about products, eligibility, process | Yes (template) | Normal |
| `faq_rates` | Questions about interest rates or fees | Yes (template + disclaimer) | Normal |
| `product_inquiry` | Specific product questions | Draft for staff | Normal |
| `application_help` | Help with loan application process | Yes (step-by-step guide) | Normal |
| `account_issue` | Problems with portal access, login | Draft for staff | High |
| `complaint` | Customer complaints | Draft for staff + acknowledge | High |
| `partnership` | Business partnership inquiries | Acknowledge + draft for staff | Normal |
| `legal_notice` | Legal correspondence or demands | No auto-response; escalate | Critical |
| `transaction_query` | Questions about specific payments/transactions | No auto-response; escalate | High |
| `spam_or_irrelevant` | Spam, phishing, or irrelevant content | No response; flag | Low |

### 6.2 Auto-Response Decision Flow

```
AUTO-RESPONSE DECISION FLOW
───────────────────────────

1. INBOUND EMAIL RECEIVED
   │
   ▼
2. CATEGORIZE EMAIL
   │  (LLM classification + keyword matching)
   │
   ├── faq_general / faq_rates / application_help
   │     │
   │     ▼
   │   3a. CHECK AUTO-RESPONSE LIMIT
   │       (max 2 auto-responses per thread)
   │       │
   │       ├── Under limit ──► Generate response ──► HITL check ──► Send
   │       └── Over limit ──► Draft for staff (stop auto-responding)
   │
   ├── product_inquiry / account_issue / complaint / partnership
   │     │
   │     ▼
   │   3b. DRAFT RESPONSE FOR STAFF
   │       Send acknowledgment to sender ("We've received your email...")
   │       Prepare draft for staff review
   │       Set thread status = STAFF_PENDING
   │
   ├── legal_notice / transaction_query
   │     │
   │     ▼
   │   3c. ESCALATE IMMEDIATELY
   │       No auto-response
   │       Set thread status = ESCALATED
   │       Notify management via HITL
   │
   └── spam_or_irrelevant
         │
         ▼
       3d. FLAG AND IGNORE
         Move to Agent_Processed folder
         Log for review
```

### 6.3 Auto-Response Templates

Auto-responses use pre-approved templates with dynamic variable substitution:

| Template | Variables | Example Use |
|----------|-----------|-------------|
| `faq_general_response` | `{{sender_name}}`, `{{faq_link}}` | General product questions |
| `faq_rates_response` | `{{sender_name}}`, `{{rates_disclaimer}}` | Rate/fee inquiries |
| `application_help_response` | `{{sender_name}}`, `{{portal_link}}`, `{{help_guide_link}}` | Application process help |
| `acknowledgment` | `{{sender_name}}`, `{{ticket_id}}`, `{{expected_response_time}}` | Acknowledge receipt for staff-handled emails |

> **Template Rule:** All auto-response templates must be pre-approved by management. The Email Agent cannot create new templates autonomously — new templates require HITL approval and are stored in the agent's configuration.

### 6.4 Send Response Tool

**Tool: `send_response`**

```json
{
  "name": "send_response",
  "description": "Send an email response in an existing thread. Can be used for auto-responses (pre-approved templates) or staff-approved draft responses.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "thread_id": {
        "type": "string",
        "description": "Thread ID to respond in"
      },
      "from_address": {
        "type": "string",
        "enum": ["info@edufin.co.ke", "support@edufin.co.ke"]
      },
      "subject": {
        "type": "string",
        "description": "Email subject (Re: prefix added automatically for replies)"
      },
      "body_html": {
        "type": "string",
        "description": "HTML email body"
      },
      "body_text": {
        "type": "string",
        "description": "Plain text fallback"
      },
      "response_type": {
        "type": "string",
        "enum": ["auto_template", "auto_generated", "staff_approved_draft"],
        "description": "Whether this is a template auto-response, LLM-generated auto-response, or staff-approved draft"
      },
      "in_reply_to": {
        "type": "string",
        "description": "Message-ID being replied to (for thread correlation)"
      }
    },
    "required": ["thread_id", "from_address", "subject", "body_html", "response_type"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "response_type is 'auto_generated' and email contains financial claims or product-specific information",
    "destructive": false,
    "idempotent": false,
    "category": "email_outbound",
    "timeout_ms": 30000,
    "retry_policy": "auto"
  }
}
```

---

## 7. HITL Triggers

### 7.1 Email-Specific HITL Conditions

| Condition | Trigger | Rationale |
|-----------|---------|-----------|
| Email campaign | `send_campaign` or `initiate_conversation` with type `campaign` | Marketing emails require management approval |
| Product announcement | Email announces a new product or offering | Content accuracy and compliance review |
| Re-engagement campaign | Email to inactive subscribers | Brand perception; avoid appearing spammy |
| LLM-generated response with claims | Auto-generated response mentions rates, terms, or approval likelihood | Regulatory compliance |
| Legal notice received | Inbound email categorized as `legal_notice` | Must be handled by legal/management, not auto-responded |
| Transaction query | Email asks about a specific payment or transaction | Requires access to financial data; must be handled by staff |
| High-volume send | Campaign with > 100 recipients | Verify content and targeting before mass send |
| Sensitive complaint | Complaint involving allegations of misconduct or regulatory violations | Must be escalated to management immediately |

### 7.2 Always-Autonomous Actions (No HITL)

| Action | Rationale |
|--------|-----------|
| `check_inbox` | Read-only; no external impact |
| `categorize_email` | Internal classification; no external impact |
| `draft_response` | Draft only; not sent to recipient |
| `get_thread` | Read-only; no external impact |
| `escalate_to_staff` | Internal routing; improves response time |
| Auto-response with pre-approved template | Template already approved by management |

---

## 8. Scheduled Workflows

### 8.1 Default Schedule

| Workflow | Schedule | Description |
|----------|----------|-------------|
| Inbox check (info@) | Every 2 minutes | Poll IMAP for new emails |
| Inbox check (support@) | Every 2 minutes | Poll IMAP for new emails |
| Follow-up check | 09:00 EAT daily | Check for threads with no response > 48 hours |
| Stale thread cleanup | 17:00 EAT Friday | Mark unresolved threads > 7 days as ESCALATED |
| Campaign schedule check | 08:00 EAT daily | Check for scheduled campaigns due today |
| Deliverability check | 06:00 EAT daily | Verify SPF/DKIM/DMARC records are valid |

### 8.2 Follow-Up Logic

The Email Agent sends follow-up messages in these scenarios:

| Scenario | Delay | Message Type | HITL Required |
|----------|-------|-------------|---------------|
| Staff hasn't responded to STAFF_PENDING thread | 48 hours | Internal reminder to staff (not to client) | No |
| Client hasn't replied to agent's question | 5 business days | Gentle follow-up to client | Conditional |
| Welcome email — no engagement | 7 days | Re-engagement with resources | Yes |
| Application started but not completed | 3 days | Reminder with help link | No (template) |

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Master Agent Architecture](./master-agent.md)
- [Technical Integration & Workflow](./integration.md)
- [Laravel Architecture](../laravel/README.md)
- [WordPress Architecture](../wordpress/README.md)
