# MCP Protocol Specification

## Inter-Agent Communication Architecture

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Protocol Overview](#1-protocol-overview)
2. [MCP Topology](#2-mcp-topology)
3. [Message Format](#3-message-format)
4. [Tool Discovery & Registration](#4-tool-discovery--registration)
5. [Resource Access](#5-resource-access)
6. [Prompt Templates](#6-prompt-templates)
7. [Transport Layer](#7-transport-layer)
8. [Security & Authentication](#8-security--authentication)

---

## 1. Protocol Overview

The EduFin AI Agents system uses the **Model Context Protocol (MCP)** — an open standard that defines how AI applications exchange context, tools, and resources. MCP standardizes the communication between the Master Agent (Host) and each specialized sub-agent (Server).

### Protocol Characteristics

| Characteristic | Specification |
|----------------|---------------|
| Base Protocol | JSON-RPC 2.0 |
| Encoding | UTF-8 JSON |
| Transport | stdio (local) / SSE (remote) |
| Connection Model | 1:1 (Host:Server) |
| Direction | Bidirectional (request/response + notifications) |
| State | Stateful sessions per connection |

### MCP Role Mapping

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       MCP ROLE MAPPING                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  MCP HOST                                                               │
│  ════════                                                               │
│  • The application that wants to access tools/resources                 │
│  • EduFin Implementation: Master Agent                                  │
│  • Initiates connections to sub-agent MCP Servers                       │
│  • Aggregates tool results and manages task lifecycle                   │
│                                                                         │
│  MCP CLIENT                                                             │
│  ══════════                                                             │
│  • Maintains 1:1 connection with a single MCP Server                    │
│  • EduFin Implementation: Master Agent connection manager              │
│  • One client instance per sub-agent connection                         │
│  • Handles protocol-level message routing                               │
│                                                                         │
│  MCP SERVER                                                             │
│  ══════════                                                             │
│  • Exposes tools, resources, and prompts via the protocol               │
│  • EduFin Implementation: Each sub-agent (Marketing, Email,            │
│    Self-Healing)                                                        │
│  • Responds to tool calls and resource requests                         │
│  • Can also act as MCP Client to external MCP Servers                   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. MCP Topology

### 2.1 Connection Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           MCP CONNECTION TOPOLOGY                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│                    ┌──────────────────────┐                                    │
│                    │                      │                                    │
│                    │    MASTER AGENT      │                                    │
│                    │    (MCP Host)        │                                    │
│                    │                      │                                    │
│                    │  ┌────────────────┐ │                                    │
│                    │  │ Connection     │ │                                    │
│                    │  │ Manager        │ │                                    │
│                    │  └──┬───┬───┬─────┘ │                                    │
│                    └─────┼───┼───┼───────┘                                    │
│                          │   │   │                                            │
│                     ┌────┘   │   └────┐                                       │
│                     │        │        │                                       │
│              ┌──────┴──┐ ┌──┴──────┐ ┌┴────────────┐                          │
│              │ MCP     │ │ MCP     │ │ MCP          │                          │
│              │ Client  │ │ Client  │ │ Client       │                          │
│              │ #1      │ │ #2      │ │ #3           │                          │
│              └──┬──────┘ └──┬──────┘ └──┬───────────┘                          │
│                 │           │           │                                      │
│          stdio/SSE    stdio/SSE    stdio/SSE                                   │
│                 │           │           │                                      │
│          ┌──────┴──┐ ┌─────┴─────┐ ┌──┴───────────┐                           │
│          │ MCP     │ │ MCP       │ │ MCP          │                           │
│          │ Server  │ │ Server    │ │ Server       │                           │
│          │ (Mktg)  │ │ (Email)   │ │ (Self-Heal)  │                           │
│          └────┬────┘ └─────┬─────┘ └──┬───────────┘                           │
│               │            │          │                                       │
│               ▼            ▼          ▼                                       │
│          External      External    External                                   │
│          Services      Services    Services                                   │
│          (X, WP)       (SMTP)      (Git, SSH)                                 │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Sub-Agent as MCP Client (Chained Connections)

Sub-agents may also act as MCP Clients to connect to external MCP Servers (e.g., a WordPress MCP Server, an X/Twitter MCP Server, an SMTP MCP Server):

```
Master Agent (MCP Host)
  │
  ├── MCP Client #1 ──► Marketing Agent (MCP Server)
  │                         │
  │                         ├── MCP Client ──► X/Twitter MCP Server
  │                         └── MCP Client ──► WordPress REST MCP Server
  │
  ├── MCP Client #2 ──► Email Agent (MCP Server)
  │                         │
  │                         └── MCP Client ──► SMTP MCP Server (info@, support@)
  │
  └── MCP Client #3 ──► Self-Healing Agent (MCP Server)
                            │
                            ├── MCP Client ──► Git MCP Server
                            └── MCP Client ──► Server Health MCP Server
```

---

## 3. Message Format

### 3.1 JSON-RPC 2.0 Envelope

All MCP messages use the JSON-RPC 2.0 envelope format.

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": "req-uuid-1234",
  "method": "tools/call",
  "params": {
    "name": "post_tweet",
    "arguments": {
      "content": "New education financing packages available!",
      "scheduled_at": "2026-08-08T14:00:00Z"
    }
  }
}
```

**Response (Success):**
```json
{
  "jsonrpc": "2.0",
  "id": "req-uuid-1234",
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Tweet posted successfully. Tweet ID: 1823456789012"
      }
    ],
    "metadata": {
      "tweet_id": "1823456789012",
      "posted_at": "2026-08-08T14:00:02Z",
      "agent": "marketing-agent"
    }
  }
}
```

**Response (Error):**
```json
{
  "jsonrpc": "2.0",
  "id": "req-uuid-1234",
  "error": {
    "code": -32000,
    "message": "X API rate limit exceeded",
    "data": {
      "retry_after": 300,
      "agent": "marketing-agent",
      "escalate": false
    }
  }
}
```

**Notification (no response expected):**
```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "task_id": "task-abc-123",
    "progress": 75,
    "status": "generating_content"
  }
}
```

### 3.2 Standard Methods

| Method | Direction | Purpose |
|--------|-----------|---------|
| `initialize` | Host → Server | Establish MCP session, negotiate capabilities |
| `tools/list` | Host → Server | Discover available tools from sub-agent |
| `tools/call` | Host → Server | Invoke a specific tool on a sub-agent |
| `resources/list` | Host → Server | List available resources |
| `resources/read` | Host → Server | Read a specific resource |
| `prompts/list` | Host → Server | List available prompt templates |
| `prompts/get` | Host → Server | Retrieve a specific prompt template |
| `notifications/progress` | Server → Host | Report task progress |
| `notifications/log` | Server → Host | Send log entries to host |
| `notifications/hitl_required` | Server → Host | Signal that human approval is needed |

---

## 4. Tool Discovery & Registration

### 4.1 Initialization Handshake

When the Master Agent connects to a sub-agent, it performs an `initialize` handshake:

```
Master Agent                          Sub-Agent (MCP Server)
     │                                         │
     │  ─── initialize ──────────────────────►  │
     │      (protocolVersion, capabilities)    │
     │                                         │
     │  ◄──────── initialize response ───────  │
     │      (serverInfo, capabilities)         │
     │                                         │
     │  ─── notifications/initialized ───────►  │
     │                                         │
     │  ─── tools/list ─────────────────────►  │
     │                                         │
     │  ◄──────── tools list response ───────  │
     │      (tool schemas)                     │
     │                                         │
     │  Session established                    │
```

### 4.2 Tool Schema Format

Each sub-agent exposes its tools with a JSON Schema definition:

```json
{
  "tools": [
    {
      "name": "post_tweet",
      "description": "Post a promotional tweet to the EduFin X/Twitter account. Requires HITL approval for sensitive content.",
      "inputSchema": {
        "type": "object",
        "properties": {
          "content": {
            "type": "string",
            "maxLength": 280,
            "description": "The tweet text content"
          },
          "media_ids": {
            "type": "array",
            "items": { "type": "string" },
            "description": "Optional media attachment IDs"
          },
          "scheduled_at": {
            "type": "string",
            "format": "date-time",
            "description": "Optional scheduled posting time (ISO 8601)"
          }
        },
        "required": ["content"]
      },
      "annotations": {
        "hitl_required": "conditional",
        "hitl_condition": "content contains financial claims or promotional offers",
        "destructive": false,
        "idempotent": false,
        "category": "social_media"
      }
    }
  ]
}
```

### 4.3 Tool Annotation Fields

| Annotation | Values | Purpose |
|------------|--------|---------|
| `hitl_required` | `never`, `conditional`, `always` | When human approval is needed |
| `hitl_condition` | String (description) | Condition that triggers HITL for conditional tools |
| `destructive` | `true` / `false` | Whether the tool has irreversible side effects |
| `idempotent` | `true` / `false` | Whether repeated calls produce the same result |
| `category` | String | Tool category for routing and auditing |
| `timeout_ms` | Integer | Maximum execution time before timeout |
| `retry_policy` | `none`, `auto`, `manual` | How failures should be handled |

---

## 5. Resource Access

Resources are read-only data sources that sub-agents expose to the Master Agent.

### 5.1 Resource URI Scheme

```
edufin://marketing/trends/current          → Current social media trends
edufin://marketing/content/scheduled       → Scheduled content calendar
edufin://email/threads/{thread_id}         → Email conversation thread
edufin://email/mailbox/{address}/inbox     → Inbox for a specific mailbox
edufin://selfhealing/health/wordpress      → WordPress health status
edufin://selfhealing/health/laravel        → Laravel health status
edufin://selfhealing/logs/{service}        → Service error logs
```

### 5.2 Resource Read Example

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": "req-uuid-5678",
  "method": "resources/read",
  "params": {
    "uri": "edufin://email/threads/thread-xyz-123"
  }
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": "req-uuid-5678",
  "result": {
    "contents": [
      {
        "uri": "edufin://email/threads/thread-xyz-123",
        "mimeType": "application/json",
        "text": "{\"thread_id\":\"thread-xyz-123\",\"subject\":\"Loan Inquiry\",\"messages\":[...]}"
      }
    ]
  }
}
```

---

## 6. Prompt Templates

Prompt templates are pre-defined task definitions that the Master Agent can retrieve and use to instruct sub-agents.

### 6.1 Available Prompt Templates

| Prompt ID | Sub-Agent | Purpose |
|-----------|-----------|---------|
| `marketing_trend_report` | Marketing Agent | Generate a weekly trend analysis report |
| `marketing_create_promotion` | Marketing Agent | Create a promotional tweet based on product data |
| `email_draft_response` | Email Agent | Draft a response to an email inquiry |
| `email_follow_up` | Email Agent | Generate a follow-up message for an existing thread |
| `selfheal_diagnose` | Self-Healing Agent | Run full diagnostic on a target system |
| `selfheal_patch` | Self-Healing Agent | Generate a patch for a detected issue |

### 6.2 Prompt Retrieval Example

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": "req-uuid-9012",
  "method": "prompts/get",
  "params": {
    "name": "marketing_create_promotion",
    "arguments": {
      "product": "education_loan",
      "audience": "parents",
      "tone": "professional"
    }
  }
}
```

---

## 7. Transport Layer

### 7.1 Transport Options

| Transport | Use Case | Characteristics |
|-----------|----------|-----------------|
| **stdio** | Master Agent ↔ Sub-Agent (same host) | Low latency, no network overhead, secure (local pipes) |
| **SSE (Server-Sent Events)** | Master Agent ↔ Sub-Agent (remote) | Supports long-running operations, streaming progress |
| **HTTP** | Sub-Agent ↔ External MCP Servers | Standard HTTP with JSON-RPC payloads |

### 7.2 Transport Selection

```
Deployment Scenario                    Transport
───────────────────                    ──────────
Master + Sub-Agents on same server     stdio
Master + Sub-Agents on different       SSE
servers
Sub-Agent → External MCP Server        HTTP
```

### 7.3 Connection Lifecycle

| Phase | Action |
|-------|--------|
| **Connect** | Master Agent opens transport channel to sub-agent |
| **Initialize** | Protocol version negotiation, capability exchange |
| **Ready** | Session is active; tools/resources available |
| **Active** | Tool calls, resource reads, notifications flowing |
| **Disconnect** | Clean shutdown via `shutdown` method, or timeout |
| **Reconnect** | Master Agent re-establishes connection with backoff |

---

## 8. Security & Authentication

### 8.1 Inter-Agent Authentication

| Layer | Mechanism | Purpose |
|-------|-----------|---------|
| Transport | TLS 1.2+ (SSE/HTTP) | Encrypt communication channel |
| Session | API key per sub-agent | Authenticate Master Agent to sub-agent |
| Tool-level | Capability tokens | Restrict which tools the Master Agent can invoke |
| Resource-level | URI ACLs | Restrict which resources are readable |

### 8.2 Secret Management

Agent credentials (API keys, SMTP passwords, OAuth tokens) are stored in the same secret management system used by the Laravel application:

| Secret | Storage | Access |
|--------|---------|--------|
| X/Twitter API credentials | Environment variables / Vault | Marketing Agent only |
| SMTP credentials (info@, support@) | Environment variables / Vault | Email Agent only |
| Git deploy keys | SSH key store | Self-Healing Agent only |
| WordPress Application Password | Environment variables / Vault | Marketing Agent only |
| Laravel API key | Environment variables / Vault | All agents (read-only endpoints) |

> **Security Rule:** No agent may access another agent's credentials. The Master Agent does not handle raw credentials — it only routes tool calls.

### 8.3 Audit Logging

Every MCP message is logged for audit purposes:

| Log Field | Description |
|-----------|-------------|
| `timestamp` | ISO 8601 timestamp |
| `message_id` | Unique message identifier |
| `session_id` | MCP session identifier |
| `direction` | `host_to_server` or `server_to_host` |
| `method` | JSON-RPC method called |
| `tool_name` | Tool invoked (if applicable) |
| `agent` | Sub-agent identifier |
| `result_status` | `success`, `error`, `hitl_pending` |
| `latency_ms` | Response time |
| `correlation_id` | Links to originating task |

---

**See Also:**
- [AI Agents Overview](./README.md)
- [Master Agent Architecture](./master-agent.md)
- [Technical Integration & Workflow](./integration.md)
