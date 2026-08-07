# Support Agent

## Customer Support Sub-Agent (Chat, WhatsApp & Email)

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Agent Overview](#1-agent-overview)
2. [MCP Server Definition](#2-mcp-server-definition)
3. [WordPress Chat Widget Integration](#3-wordpress-chat-widget-integration)
4. [WhatsApp Support via WAHA](#4-whatsapp-support-via-waha)
5. [Email Support Integration](#5-email-support-integration)
6. [Conversation State Management](#6-conversation-state-management)
7. [HITL Triggers](#7-hitl-triggers)
8. [Scheduled Workflows](#8-scheduled-workflows)

---

## 1. Agent Overview

The Support Agent is a specialized sub-agent responsible for **real-time customer support** across EduFin's support channels. It powers the WordPress site support chat widget, handles WhatsApp support conversations via WAHA, and monitors/responds to support emails. It operates as an MCP Server, exposing tools that the Master Agent can invoke.

The agent's primary goal is to provide fast, accurate responses to common customer inquiries (FAQs, product questions, application help) while escalating complex or sensitive issues to human support staff. It bridges the gap between self-service content and human support, ensuring visitors get immediate assistance 24/7.

### Scope Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        SUPPORT AGENT - SCOPE                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  IN SCOPE:                                                                      │
│  ✓ Power the WordPress site support chat widget (bottom-right corner)          │
│  ✓ Real-time chat responses to website visitor questions                        │
│  ✓ WhatsApp support conversations via WAHA server                               │
│  ✓ Monitor and respond to customer_care@edufin.co.ke and support@edufin.co.ke │
│  ✓ Answer FAQs about EduFin products, eligibility, application process         │
│  ✓ Categorize and escalate complex support issues to human staff               │
│  ✓ Collect visitor contact information for follow-up                           │
│  ✓ Provide links to relevant WordPress pages (FAQs, blog posts, product pages) │
│  ✓ Track support conversation state and history                                │
│  ✓ Handoff conversations to human support staff when needed                    │
│                                                                                 │
│  OUT OF SCOPE:                                                                  │
│  ✗ Accessing Laravel user accounts or loan data directly                       │
│  ✗ Processing loan applications or financial transactions                      │
│  ✗ Modifying WordPress content or theme                                        │
│  ✗ Handling legal notices (escalate to HITL)                                   │
│  ✗ Sending unsolicited marketing messages (that's the Marketing Agent's job)   │
│  ✗ Auto-responding to messages containing financial transaction details        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Managed Channels

| Channel | Address/Location | Purpose |
|---------|------------------|---------|
| WordPress Chat Widget | Bottom-right of edufin.co.ke | Real-time visitor support |
| WhatsApp | Via WAHA server (business number) | WhatsApp customer support |
| Email | `customer_care@edufin.co.ke` | Customer care inquiries |
| Email | `support@edufin.co.ke` | Technical support, account issues |

### External Service Dependencies

| Service | Purpose | Auth Method |
|---------|---------|-------------|
| WordPress Chat Widget API | Real-time chat via WebSocket | API Key + WebSocket token |
| WAHA Server | WhatsApp message send/receive | API Key |
| SMTP Server (outbound) | Send support emails | SMTP Auth (username + password) |
| IMAP Server (inbound) | Read support emails | IMAP Auth (username + password) |
| Laravel REST API | Fetch FAQ content, product info (read-only) | API Key (`X-API-Key`) |

---

## 2. MCP Server Definition

### 2.1 Server Metadata

```json
{
  "serverInfo": {
    "name": "edufin-support-agent",
    "version": "1.0.0",
    "description": "Support sub-agent for WordPress chat widget, WhatsApp, and email customer support",
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
| `handle_chat_message` | Process an incoming chat widget message and generate a response | Conditional | No |
| `send_chat_response` | Send a response to the WordPress chat widget | Conditional | No |
| `escalate_to_staff` | Transfer a chat/WhatsApp/email conversation to human staff | Never | No |
| `send_whatsapp_message` | Send a WhatsApp message via WAHA server | Conditional | No |
| `receive_whatsapp_message` | Process an incoming WhatsApp message via WAHA webhook | Conditional | No |
| `check_support_inbox` | Fetch and categorize new support emails | Never | No |
| `draft_support_response` | Generate a draft response for a support email | Never | No |
| `send_support_email` | Send a support email response | Conditional | No |
| `get_faq_content` | Fetch FAQ content from WordPress/Laravel for answering questions | Never | No |
| `get_product_info` | Fetch product info from Laravel API for answering questions | Never | No |
| `collect_contact_info` | Collect visitor contact information for follow-up | Never | No |
| `get_conversation_history` | Retrieve full conversation history for a visitor/client | Never | No |

### 2.3 Exposed Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://support/conversations/active` | JSON | All active support conversations |
| `edufin://support/conversations/{id}` | JSON | Full conversation history by ID |
| `edufin://support/faq/answers` | JSON | FAQ knowledge base |
| `edufin://support/escalations/pending` | JSON | Conversations escalated to staff |
| `edufin://support/whatsapp/sessions` | JSON | Active WhatsApp support sessions |

### 2.4 Exposed Prompts

| Prompt Name | Arguments | Purpose |
|-------------|-----------|---------|
| `support_faq_response` | `question`, `channel` | Generate an FAQ response for chat/WhatsApp/email |
| `support_escalation_summary` | `conversation_id`, `reason` | Generate a summary for staff handoff |
| `support_welcome_message` | `channel`, `visitor_name` | Generate a welcome message for new chat sessions |

### 2.5 Key Tool Schemas

**Tool: `handle_chat_message`**

```json
{
  "name": "handle_chat_message",
  "description": "Process an incoming chat widget message from a website visitor and generate an appropriate response. Handles FAQ lookups, product info queries, contact collection, and escalation detection.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "conversation_id": {
        "type": "string",
        "description": "Unique conversation ID for the chat session"
      },
      "visitor_id": {
        "type": "string",
        "description": "Anonymous visitor identifier (from chat widget session)"
      },
      "message": {
        "type": "string",
        "description": "The visitor's message content"
      },
      "visitor_name": {
        "type": "string",
        "description": "Visitor name if provided (optional for anonymous visitors)"
      },
      "visitor_contact": {
        "type": "object",
        "description": "Contact info collected from the visitor (if provided)",
        "properties": {
          "email": { "type": "string", "format": "email" },
          "phone": { "type": "string" },
          "name": { "type": "string" }
        }
      },
      "message_number": {
        "type": "integer",
        "description": "Sequential message number in the conversation (for rate limiting and auto-response tracking)",
        "minimum": 1
      }
    },
    "required": ["conversation_id", "visitor_id", "message"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "message contains financial transaction details, legal notice, regulatory complaint, or specific account/loan information request",
    "destructive": false,
    "idempotent": false,
    "category": "chat_inbound",
    "timeout_ms": 15000
  }
}
```

**Tool: `send_whatsapp_message`**

```json
{
  "name": "send_whatsapp_message",
  "description": "Send a WhatsApp message to a contact via the WAHA (WhatsApp HTTP API) server. Only sends to known/opted-in contacts. Requires HITL approval for messages containing financial claims or sensitive account information.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "conversation_id": {
        "type": "string",
        "description": "Conversation ID for tracking and state management"
      },
      "phone_number": {
        "type": "string",
        "description": "Recipient phone number in international format (e.g., 254712345678)"
      },
      "message": {
        "type": "string",
        "description": "Text message content to send"
      },
      "media_url": {
        "type": "string",
        "description": "Optional URL for media attachment (image, document) to send via WAHA"
      },
      "media_type": {
        "type": "string",
        "enum": ["image", "document", "audio"],
        "description": "Type of media attachment (required if media_url is provided)"
      },
      "response_type": {
        "type": "string",
        "enum": ["auto_template", "auto_generated", "staff_approved_draft"],
        "description": "Whether this is a template auto-response, LLM-generated response, or staff-approved draft"
      }
    },
    "required": ["conversation_id", "phone_number", "message", "response_type"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "response_type is 'auto_generated' and message contains financial claims, rates, or account-specific information",
    "destructive": false,
    "idempotent": false,
    "category": "whatsapp_outbound",
    "timeout_ms": 15000,
    "retry_policy": "auto"
  }
}
```

**Tool: `send_support_email`**

```json
{
  "name": "send_support_email",
  "description": "Send a support email response from an EduFin support mailbox. Can be used for auto-responses (pre-approved templates) or staff-approved draft responses.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "thread_id": {
        "type": "string",
        "description": "Thread ID to respond in"
      },
      "from_address": {
        "type": "string",
        "enum": ["customer_care@edufin.co.ke", "support@edufin.co.ke"],
        "description": "Sending mailbox"
      },
      "to_address": {
        "type": "string",
        "format": "email",
        "description": "Recipient email address"
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
        "description": "Plain text fallback body"
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
    "required": ["thread_id", "from_address", "to_address", "subject", "body_html", "response_type"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "response_type is 'auto_generated' and email contains financial claims or account-specific information",
    "destructive": false,
    "idempotent": false,
    "category": "email_outbound",
    "timeout_ms": 30000,
    "retry_policy": "auto"
  }
}
```

**Tool: `escalate_to_staff`**

```json
{
  "name": "escalate_to_staff",
  "description": "Transfer a support conversation (chat, WhatsApp, or email) to human support staff. Generates an escalation summary, notifies staff via the HITL interface, and updates the conversation state to ESCALATED or STAFF_PENDING.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "conversation_id": {
        "type": "string",
        "description": "ID of the conversation to escalate"
      },
      "channel": {
        "type": "string",
        "enum": ["chat", "whatsapp", "email"],
        "description": "Channel the conversation originated from"
      },
      "reason": {
        "type": "string",
        "enum": ["complex_issue", "complaint", "legal_notice", "transaction_query", "account_access", "regulatory", "sentiment_negative", "auto_response_limit", "staff_request"],
        "description": "Reason for escalation"
      },
      "priority": {
        "type": "string",
        "enum": ["normal", "high", "critical"],
        "description": "Escalation priority level",
        "default": "normal"
      },
      "summary": {
        "type": "string",
        "description": "Optional pre-generated escalation summary. If omitted, the agent generates one using the support_escalation_summary prompt."
      },
      "visitor_contact": {
        "type": "object",
        "description": "Contact information for the visitor/client (if collected)",
        "properties": {
          "email": { "type": "string", "format": "email" },
          "phone": { "type": "string" },
          "name": { "type": "string" }
        }
      }
    },
    "required": ["conversation_id", "channel", "reason"]
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": true,
    "category": "escalation",
    "timeout_ms": 10000
  }
}
```

---

## 3. WordPress Chat Widget Integration

### 3.1 Widget Overview

The support chat widget is embedded on the EduFin WordPress site (`edufin.co.ke`) in the **bottom-right corner** of every page. It is loaded via a JavaScript snippet injected into the WordPress theme footer, providing a lightweight, real-time chat interface for site visitors.

| Feature | Description |
|---------|-------------|
| **Placement** | Bottom-right corner, all WordPress pages |
| **Loading** | JavaScript snippet in WordPress theme footer |
| **Authentication** | Anonymous visitors (no login required) |
| **Contact Collection** | Can collect name, email, phone for follow-up |
| **Online/Offline Status** | Widget shows agent availability status |
| **Rate Limiting** | Max 10 messages per minute per visitor |

### 3.2 WebSocket Communication

The chat widget communicates with the Support Agent backend via a persistent WebSocket connection:

```
CHAT WIDGET COMMUNICATION FLOW
───────────────────────────────

  ┌──────────────────┐         WebSocket (wss://)          ┌──────────────────────┐
  │  WordPress       │ ◄─────────────────────────────────► │  Support Agent       │
  │  Chat Widget     │                                      │  Backend             │
  │  (Browser)       │                                      │  (MCP Server)        │
  └────────┬─────────┘                                      └──────────┬───────────┘
           │                                                           │
           │  1. Visitor opens site → widget loads                     │
           │  2. Widget establishes WebSocket connection               │
           │  3. Visitor sends message → handle_chat_message           │
           │  4. Agent processes → generates response                  │
           │  5. Agent sends response → send_chat_response             │
           │  6. Widget displays response to visitor                   │
           │                                                           │
           │  If agent offline or escalated:                           │
           │  → Widget shows contact collection form                   │
           │  → collect_contact_info stores visitor details            │
           └───────────────────────────────────────────────────────────┘
```

| Parameter | Value |
|-----------|-------|
| Protocol | WebSocket Secure (`wss://`) |
| Endpoint | `wss://edufin.co.ke/support-ws` |
| Authentication | API Key + WebSocket token (per-session) |
| Heartbeat Interval | 30 seconds |
| Reconnection | Auto-reconnect with exponential backoff |

### 3.3 Online/Offline Behavior

The widget adapts its behavior based on agent availability:

| Agent State | Widget Behavior |
|-------------|-----------------|
| **Online** | Shows chat input; agent responds in real-time |
| **Offline** | Shows contact collection form (name, email, phone, message) |
| **Escalated** | Shows "Transferring to a human agent..." then contact form if no staff available |

When the agent is available (online), it responds to visitor messages in real-time using FAQ content, product info, and conversation context. When the agent is offline or a conversation is escalated to staff, the widget displays a form to collect visitor contact information for follow-up.

### 3.4 Rate Limiting & Abuse Prevention

| Limit | Value | Rationale |
|-------|-------|-----------|
| Max messages per minute | 10 per visitor | Prevent spam and abuse |
| Max messages per session | 100 | Prevent prolonged automated conversations |
| Max conversation duration | 30 minutes | Encourage efficient resolution |
| IP-based throttling | 50 messages/hour per IP | Prevent multi-session abuse |

> **Security Note:** The chat widget uses anonymous visitor sessions (no login required). However, the agent can collect name, email, and phone voluntarily for follow-up purposes. No PII is stored without the visitor's explicit consent.

---

## 4. WhatsApp Support via WAHA

### 4.1 WAHA Server Overview

The Support Agent integrates with [WAHA (WhatsApp HTTP API)](https://waha.devlike.pro) to provide customer support over WhatsApp. WAHA exposes a REST API that wraps the WhatsApp Web protocol, enabling programmatic send/receive of WhatsApp messages.

| Parameter | Value |
|-----------|-------|
| WAHA Base URL | `http://waha.edufin.co.ke` |
| Auth Method | API Key (header: `Authorization: Bearer <key>`) |
| WhatsApp Business Number | Configured in WAHA session |
| Session Name | `edufin-support` |

### 4.2 Inbound Message Flow

WAHA receives incoming WhatsApp messages and forwards them to the Support Agent via webhook:

```
WHATSAPP INBOUND FLOW
─────────────────────

  ┌──────────────┐     ┌──────────────┐     ┌──────────────────────┐
  │  Customer    │ ──► │  WhatsApp    │ ──► │  WAHA Server         │
  │  (WhatsApp)  │     │  Network     │     │  (waha.edufin.co.ke) │
  └──────────────┘     └──────────────┘     └──────────┬───────────┘
                                                       │
                                          Webhook POST │
                                          /api/webhook │
                                                       ▼
                                          ┌──────────────────────┐
                                          │  Support Agent       │
                                          │  receive_whatsapp_   │
                                          │  message()           │
                                          └──────────────────────┘
```

### 4.3 Outbound Message Flow

The Support Agent sends WhatsApp messages via the WAHA REST API:

| API Endpoint | Method | Purpose |
|--------------|--------|---------|
| `POST /api/sendText` | POST | Send a text message |
| `POST /api/sendImage` | POST | Send an image message |
| `POST /api/sendFile` | POST | Send a document (PDF, etc.) |
| `POST /api/webhook/delivery` | Webhook | Delivery report callback |

### 4.4 WhatsApp Support Rules

| Rule | Description |
|------|-------------|
| **Opt-in required** | Only responds to messages from known/opted-in contacts |
| **Session tracking** | WhatsApp conversations tracked same as chat widget conversations |
| **Media support** | Can handle media messages (images, documents) via WAHA |
| **Delivery reports** | WAHA webhook (`POST /api/webhook/delivery`) confirms delivery |
| **Business hours** | Auto-responds 24/7 for FAQs; escalates to staff outside business hours for complex issues |
| **Auto-response limit** | Max 5 auto-responses per conversation before escalating to staff |

> **Security Note:** The WhatsApp business number is configured exclusively in WAHA and is not shared with other agents. The Support Agent only sends messages to contacts who have opted in to WhatsApp support, complying with WhatsApp Business messaging policies.

---

## 5. Email Support Integration

### 5.1 Monitored Mailboxes

The Support Agent monitors two dedicated support mailboxes via IMAP and sends responses via SMTP:

| Mailbox | Purpose | Agent Role |
|---------|---------|------------|
| `customer_care@edufin.co.ke` | Customer care inquiries, general support | Monitor, auto-respond to FAQs, draft replies for staff |
| `support@edufin.co.ke` | Technical support, account issues, complaints | Monitor, categorize, draft replies, escalate urgent issues |

### 5.2 SMTP Configuration (Outbound)

| Parameter | Value (Example) | Notes |
|-----------|-----------------|-------|
| SMTP Host | `mail.edufin.co.ke` | EduFin mail server |
| SMTP Port | `587` | STARTTLS |
| Encryption | `TLS` | Transport-level encryption |
| Auth Method | `LOGIN` | Standard SMTP auth |
| customer_care@ Credentials | `customer_care@edufin.co.ke` / `<password>` | Stored in Vault/env |
| support@ Credentials | `support@edufin.co.ke` / `<password>` | Stored in Vault/env |
| From Address | `customer_care@edufin.co.ke` or `support@edufin.co.ke` | Per mailbox context |
| Reply-To | Same as From | Responses route back to monitored mailbox |

### 5.3 IMAP Configuration (Inbound)

| Parameter | Value | Notes |
|-----------|-------|-------|
| IMAP Host | `mail.edufin.co.ke` | Same mail server |
| IMAP Port | `993` | SSL |
| Encryption | `SSL` | Encrypted inbound |
| Polling Interval | Every 2 minutes | Configurable per mailbox |
| Folders Monitored | `INBOX` | Primary inbox per mailbox |
| Processed Folder | `Agent_Processed` | Emails moved here after processing |
| Escalated Folder | `Agent_Escalated` | Emails requiring staff attention |

> **Note:** The SMTP/IMAP configuration follows the same pattern as the Email Agent (see [Email Agent SMTP Integration](./email-agent.md#3-smtp-integration-requirements)). The Support Agent uses dedicated credentials for `customer_care@edufin.co.ke` and `support@edufin.co.ke`, separate from the Email Agent's `info@` and `support@` credentials.

### 5.4 Email Categorization

Inbound support emails are categorized to determine the appropriate response strategy:

| Category | Description | Auto-Response | Priority |
|----------|-------------|---------------|----------|
| `faq_general` | General questions about products, eligibility, process | Yes (template) | Normal |
| `account_issue` | Problems with portal access, login, account settings | Draft for staff | High |
| `complaint` | Customer complaints | Draft for staff + acknowledge | High |
| `technical_issue` | Technical problems with the website or portal | Draft for staff + acknowledge | High |
| `application_help` | Help with loan application process | Yes (step-by-step guide) | Normal |
| `legal_notice` | Legal correspondence or demands | No auto-response; escalate | Critical |
| `transaction_query` | Questions about specific payments/transactions | No auto-response; escalate | High |
| `spam_or_irrelevant` | Spam, phishing, or irrelevant content | No response; flag | Low |

### 5.5 Response Strategy

```
SUPPORT EMAIL RESPONSE FLOW
───────────────────────────

1. INBOUND EMAIL RECEIVED (customer_care@ or support@)
   │
   ▼
2. CATEGORIZE EMAIL
   │  (LLM classification + keyword matching)
   │
   ├── faq_general / application_help
   │     │
   │     ▼
   │   3a. AUTO-RESPOND WITH TEMPLATE
   │       Send pre-approved FAQ response
   │       Set thread status = AUTO_RESPONDING → RESOLVED
   │
   ├── account_issue / complaint / technical_issue
   │     │
   │     ▼
   │   3b. DRAFT RESPONSE FOR STAFF
   │       Send acknowledgment to sender
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

> **Template Rule:** All auto-response templates must be pre-approved by management. The Support Agent cannot create new templates autonomously — new templates require HITL approval and are stored in the agent's configuration.

---

## 6. Conversation State Management

### 6.1 Conversation Data Model

Every support conversation — whether from chat widget, WhatsApp, or email — is tracked with persistent state:

```json
{
  "conversation_id": "conv-uuid-abcd",
  "channel": "chat",
  "status": "ACTIVE",
  "visitor_id": "anon-visitor-123",
  "visitor_contact": {
    "name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+254712345678"
  },
  "messages": [
    {
      "message_id": "msg-uuid-001",
      "direction": "inbound",
      "channel": "chat",
      "content": "Hi, what are the eligibility requirements for a loan?",
      "timestamp": "2026-08-08T10:00:00Z",
      "sender": "visitor"
    },
    {
      "message_id": "msg-uuid-002",
      "direction": "outbound",
      "channel": "chat",
      "content": "Hello! To be eligible for an EduFin loan, you need to...",
      "timestamp": "2026-08-08T10:00:05Z",
      "sender": "support-agent",
      "response_type": "auto_generated"
    }
  ],
  "auto_response_count": 1,
  "category": "faq_general",
  "priority": "normal",
  "staff_escalated": false,
  "linked_conversations": [],
  "created_at": "2026-08-08T10:00:00Z",
  "last_activity_at": "2026-08-08T10:00:05Z",
  "metadata": {
    "source_page": "https://edufin.co.ke/eligibility",
    "visitor_ip": "196.201.220.1"
  }
}
```

### 6.2 Conversation State Machine

```
┌─────────────────────────────────────────────────────────────────────────┐
│                   SUPPORT CONVERSATION STATE MACHINE                     │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────┐                                                          │
│   │  ACTIVE  │  Conversation is open; visitor/client is engaged        │
│   └────┬─────┘                                                          │
│        │                                                                │
│        ├── New message (FAQ-eligible) ──► Evaluate response strategy     │
│        │                                                                │
│        ▼                                                                │
│   ┌──────────────────┐                                                  │
│   │ AUTO_RESPONDING  │  Agent generating/sending automated response    │
│   └──────┬───────────┘                                                  │
│          │                                                              │
│          ├── Response sent, awaiting reply ──► ACTIVE                   │
│          │                                                              │
│          ├── Cannot auto-respond ──► STAFF_PENDING                     │
│          │                                                              │
│          ├── HITL required ──► ESCALATED                               │
│          │                                                              │
│          └── Exceeds 5 auto-responses ──► ESCALATED                    │
│                                                                         │
│   ┌──────────────┐                                                      │
│   │STAFF_PENDING │  Awaiting staff response (draft may be prepared)    │
│   └──────┬───────┘                                                      │
│          │                                                              │
│          ├── Staff responds ──► RESOLVED                               │
│          │                                                              │
│          └── No response > 24h ──► ESCALATED                           │
│                                                                         │
│   ┌───────────┐                                                        │
│   │ ESCALATED │  Urgent staff attention; escalated to human support   │
│   └─────┬─────┘                                                        │
│         │                                                              │
│         ├── Staff resolves ──► RESOLVED                               │
│         │                                                              │
│         └── Staff re-engages agent ──► ACTIVE                          │
│                                                                         │
│   ┌──────────┐                                                         │
│   │ RESOLVED │  Conversation complete; no further action needed       │
│   └──────────┘                                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 6.3 Cross-Channel Conversation Correlation

Conversations can span multiple channels (chat, WhatsApp, email). If a visitor provides their email or phone number, conversations across channels can be linked:

| Correlation Key | Channels Linked | Method |
|-----------------|-----------------|--------|
| Email address | Chat ↔ Email | Visitor provides email in chat; matches inbound email sender |
| Phone number | Chat ↔ WhatsApp | Visitor provides phone in chat; matches WhatsApp contact number |
| Phone + Email | All channels | Full correlation across chat, WhatsApp, and email |

When conversations are correlated, the agent has access to the full conversation history across all channels, enabling seamless support continuity (e.g., a visitor who started in chat and followed up via email).

---

## 7. HITL Triggers

### 7.1 Support-Specific HITL Conditions

| Condition | Trigger | Rationale |
|-----------|---------|-----------|
| Complaints involving regulatory violations or misconduct allegations | Message categorized as `complaint` with regulatory keywords | Must be escalated to management immediately; legal/compliance risk |
| Messages containing financial transaction details | Message references specific payments, M-Pesa codes, or transaction IDs | Requires access to financial data; must be handled by staff; do not auto-respond |
| Legal notices | Message categorized as `legal_notice` | Must be handled by legal/management, not auto-responded |
| Messages requesting specific account/loan information | Visitor asks about their specific loan status, account balance, or application status | Conditional — verify identity first; if identity cannot be verified, escalate to staff |
| Negative sentiment detected | LLM sentiment analysis returns negative sentiment with frustration/anger indicators | Conditional — if sentiment is strongly negative, escalate to staff for empathetic handling |
| Conversation exceeds 5 auto-responses without resolution | `auto_response_count > 5` in conversation state | Escalate to staff; visitor likely has a complex issue not solvable via FAQ |

### 7.2 Always-Autonomous Actions (No HITL)

| Action | Rationale |
|--------|-----------|
| `check_support_inbox` | Read-only; no external impact |
| `draft_support_response` | Draft only; not sent to recipient |
| `get_faq_content` | Read-only; internal data fetch |
| `get_product_info` | Read-only; internal data fetch |
| `get_conversation_history` | Read-only; no external impact |
| `collect_contact_info` | Internal storage; no external communication |
| `escalate_to_staff` | Internal routing; improves response time |
| FAQ responses with pre-approved templates | Template already approved by management |
| Welcome messages | Non-substantive greeting; no financial claims |
| Fetching FAQ/product info (read-only) | Read-only; no external impact |

---

## 8. Scheduled Workflows

### 8.1 Default Schedule

| Workflow | Schedule | Description |
|----------|----------|-------------|
| Chat widget health check | Every 1 min | Verify WebSocket connection is active |
| WhatsApp session check | Every 5 min | Verify WAHA session is connected |
| Support inbox check (customer_care@) | Every 2 min | Poll IMAP for new emails |
| Support inbox check (support@) | Every 2 min | Poll IMAP for new emails |
| Follow-up check | 09:00 EAT daily | Check unresolved conversations > 24 hours |
| FAQ content refresh | 06:00 EAT daily | Refresh FAQ cache from WordPress/Laravel |
| Stale conversation cleanup | 17:00 EAT Friday | Mark unresolved conversations > 7 days as ESCALATED |

### 8.2 Follow-Up Logic

The Support Agent sends follow-up messages in these scenarios:

| Scenario | Delay | Message Type | HITL Required |
|----------|-------|-------------|---------------|
| Staff hasn't responded to STAFF_PENDING conversation | 24 hours | Internal reminder to staff (not to client) | No |
| Visitor hasn't replied to agent's question (chat) | 1 hour | Gentle follow-up in chat (if still connected) | No (template) |
| Client hasn't replied to support email | 3 business days | Gentle follow-up email | Conditional |
| Unresolved conversation > 24 hours | 09:00 EAT daily | Flag for staff follow-up | No |
| Unresolved conversation > 7 days | 17:00 EAT Friday | Mark as ESCALATED; notify management | No |

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Master Agent Architecture](./master-agent.md)
- [Marketing Agent](./marketing-agent.md)
- [Email Agent](./email-agent.md)
- [Self-Healing Agent](./self-healing-agent.md)
- [WordPress Architecture](../wordpress/README.md)
- [Technical Integration & Workflow](./integration.md)
