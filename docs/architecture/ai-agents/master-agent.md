# Master Agent Architecture

## Orchestration & Human-in-the-Loop Protocol

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Master Agent Overview](#1-master-agent-overview)
2. [Orchestration Logic](#2-orchestration-logic)
3. [Task Lifecycle Management](#3-task-lifecycle-management)
4. [Human-in-the-Loop (HITL) Protocol](#4-human-in-the-loop-hitl-protocol)
5. [Anomaly Detection & Escalation](#5-anomaly-detection--escalation)
6. [State Management](#6-state-management)
7. [Supervision & Health Monitoring](#7-supervision--health-monitoring)

---

## 1. Master Agent Overview

The Master Agent is the **central orchestrator** of the EduFin AI Agents system. It acts as the MCP Host, maintaining connections to all sub-agents, routing tasks, aggregating results, and enforcing the Human-in-the-Loop protocol.

### Core Responsibilities

| Responsibility | Description |
|----------------|-------------|
| **Task Intake** | Receives tasks from triggers (scheduled, event-driven, manual) |
| **Task Routing** | Determines which sub-agent(s) should handle a task |
| **Execution Supervision** | Monitors sub-agent execution, enforces timeouts |
| **Result Aggregation** | Collects and synthesizes results from sub-agents |
| **HITL Enforcement** | Detects scenarios requiring human approval; pauses execution |
| **Escalation Management** | Routes unresolved or anomalous cases to management |
| **Audit Logging** | Records full execution trace for every task |
| **Sub-Agent Health** | Monitors sub-agent availability and reconnects as needed |

### Architectural Position

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         MASTER AGENT - INTERNAL ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                         TASK INTAKE LAYER                                │   │
│  │                                                                         │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐               │   │
│  │  │ Scheduled│  │ Event    │  │ Manual   │  │ External │               │   │
│  │  │ Triggers │  │ Triggers │  │ Triggers │  │ Webhooks │               │   │
│  │  │ (cron)   │  │ (hooks)  │  │ (UI/API) │  │ (CBS,WP) │               │   │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘               │   │
│  │       └───────────────┴──────────────┴──────────────┘                   │   │
│  │                              │                                          │   │
│  │                              ▼                                          │   │
│  │                    ┌───────────────┐                                   │   │
│  │                    │  Task Queue   │                                   │   │
│  │                    │  (Redis)      │                                   │   │
│  │                    └───────┬───────┘                                   │   │
│  └────────────────────────────┼──────────────────────────────────────────┘   │
│                               │                                                │
│  ┌────────────────────────────▼──────────────────────────────────────────┐   │
│  │                      ORCHESTRATION ENGINE                               │   │
│  │                                                                         │   │
│  │  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐              │   │
│  │  │ Task Router   │  │ HITL Evaluator│  │ Result        │              │   │
│  │  │ (determines   │  │ (checks if   │  │ Aggregator    │              │   │
│  │  │  target agent)│  │  human needed)│  │ (synthesizes) │              │   │
│  │  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘              │   │
│  │          │                  │                  │                       │   │
│  │          └──────────┬───────┴──────────────────┘                       │   │
│  │                     │                                                  │   │
│  │                     ▼                                                  │   │
│  │           ┌───────────────────┐                                       │   │
│  │           │ Execution State   │                                       │   │
│  │           │ Machine           │                                       │   │
│  │           └───────────────────┘                                       │   │
│  └─────────────────────────────┬─────────────────────────────────────────┘   │
│                                │                                              │
│  ┌─────────────────────────────▼─────────────────────────────────────────┐   │
│  │                    MCP CONNECTION MANAGER                              │   │
│  │                                                                         │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐             │   │
│  │  │ MCP      │  │ MCP      │  │ MCP      │  │ MCP      │             │   │
│  │  │ Client   │  │ Client   │  │ Client   │  │ Client   │             │   │
│  │  │ #1 (Mktg)│  │ #2(Email)│  │ #3(Supp) │  │ #4(Heal) │             │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘             │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │                      AUDIT & OBSERVABILITY                              │   │
│  │                                                                         │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐              │   │
│  │  │ Action   │  │ Decision │  │ HITL     │  │ Metrics  │              │   │
│  │  │ Logger   │  │ Log      │  │ Log      │  │ Collector│              │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘              │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Orchestration Logic

### 2.1 Task Intake & Classification

Tasks enter the Master Agent through four channels:

| Channel | Trigger Source | Example |
|---------|---------------|---------|
| **Scheduled** | Cron-based timer | "Run weekly marketing trend analysis every Monday 8:00 AM" |
| **Event-Driven** | System event hook | "New inquiry submitted on WordPress contact form" |
| **Manual** | Management dashboard / API | "Post a promotional tweet about the new loan product" |
| **External Webhook** | External system callback | "Server health check failed — trigger self-healing" |

### 2.2 Task Routing Algorithm

The Master Agent routes tasks to sub-agents based on task type, agent capabilities, and current agent load:

```
TASK ROUTING DECISION FLOW
──────────────────────────

1. RECEIVE TASK
   │
   ▼
2. CLASSIFY TASK TYPE
   │
   ├── marketing ──────► Route to Marketing Agent
   ├── email ──────────► Route to Email Agent
   ├── support ────────► Route to Support Agent
   ├── infrastructure ─► Route to Self-Healing Agent
   ├── cross-domain ───► Decompose into sub-tasks (Step 3)
   └── unknown ────────► Flag for HITL review
   │
   ▼
3. DECOMPOSE (if cross-domain)
   │
   ├── Sub-task A ──► Marketing Agent
   ├── Sub-task B ──► Email Agent
   ├── Sub-task C ──► Support Agent
   └── Sub-task D ──► Self-Healing Agent
   │
   ▼
4. CHECK AGENT AVAILABILITY
   │
   ├── Available ──► Dispatch task
   ├── Busy ──────► Queue task for agent
   └── Offline ───► Escalate to HITL (agent unavailable)
   │
   ▼
5. EVALUATE HITL REQUIREMENT
   │
   ├── No HITL needed ──► Execute immediately
   ├── Conditional HITL ─► Evaluate condition, then decide
   └── Always HITL ─────► Pause and request approval
```

### 2.3 Sub-Agent Dispatch

When dispatching a task to a sub-agent, the Master Agent:

1. **Selects the appropriate MCP tool** from the sub-agent's registered tool list
2. **Constructs the tool call** with arguments derived from the task
3. **Sets a timeout** based on the tool's `timeout_ms` annotation
4. **Sends the MCP `tools/call` request** via the appropriate MCP Client
5. **Monitors for progress notifications** (`notifications/progress`)
6. **Waits for the response** or timeout

### 2.4 Result Aggregation

For multi-step or cross-domain tasks, the Master Agent aggregates results:

```
MULTI-STEP TASK EXAMPLE: "Promote new loan product across channels"
──────────────────────────────────────────────────────────────────

Step 1: Marketing Agent → analyze_trends()
  Result: { "trending_topics": ["education financing", "school fees"], ... }
  │
  ▼
Step 2: Marketing Agent → generate_content()
  Input:  trend data + product info
  Result: { "tweet_content": "...", "blog_draft": "...", ... }
  │
  ▼
Step 3: HITL CHECK → Content contains promotional claims
  Action: PAUSE → Request management approval
  │
  ▼ (approved)
  │
Step 4: Marketing Agent → post_tweet()
  Result: { "tweet_id": "1823...", "posted_at": "..." }
  │
  ▼
Step 5: Email Agent → send_campaign()
  Input:  approved content + subscriber list
  Result: { "emails_sent": 1247, "delivered": 1230, ... }
  │
  ▼
Step 6: Master Agent → Aggregate & Log
  Final Result: { "task_id": "...", "status": "completed", "steps": [...] }
```

---

## 3. Task Lifecycle Management

### 3.1 Task State Machine

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TASK STATE MACHINE                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────┐                                                          │
│   │ PENDING  │  Task received, not yet processed                       │
│   └────┬─────┘                                                          │
│        │                                                                │
│        ▼                                                                │
│   ┌──────────┐     ┌───────────┐                                       │
│   │ ROUTING  │────►│ QUEUED    │  Routed to agent, waiting for slot   │
│   └────┬─────┘     └─────┬─────┘                                       │
│        │                 │                                              │
│        ▼                 ▼                                              │
│   ┌──────────┐     ┌───────────┐                                       │
│   │ DISPATCH │     │ DISPATCH  │  Sent to sub-agent via MCP           │
│   └────┬─────┘     └─────┬─────┘                                       │
│        │                 │                                              │
│        └────────┬────────┘                                              │
│                 ▼                                                        │
│   ┌──────────────┐                                                      │
│   │ HITL_PENDING │  Awaiting human approval (if required)              │
│   └──────┬───────┘                                                      │
│          │                                                              │
│     ┌────┴────┐                                                         │
│     │         │                                                         │
│     ▼         ▼                                                         │
│  APPROVED  REJECTED                                                     │
│     │         │                                                         │
│     ▼         ▼                                                         │
│   ┌──────────┐  ┌───────────┐                                          │
│   │EXECUTING │  │ CANCELLED │  Rejected by management                  │
│   └────┬─────┘  └───────────┘                                          │
│        │                                                                │
│        ▼                                                                │
│   ┌──────────┐     ┌───────────┐                                       │
│   │COMPLETED │     │   FAILED  │  Agent could not complete task        │
│   └──────────┘     └─────┬─────┘                                       │
│                         │                                                │
│                         ▼                                                │
│                   ┌───────────┐                                         │
│                   │ ESCALATED │  Routed to management for review       │
│                   └───────────┘                                         │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Task Record Schema

Every task is persisted with a complete execution record:

```json
{
  "task_id": "task-uuid-1234",
  "parent_task_id": null,
  "trigger_source": "scheduled",
  "task_type": "marketing_promotion",
  "priority": "normal",
  "status": "completed",
  "created_at": "2026-08-08T08:00:00Z",
  "dispatched_at": "2026-08-08T08:00:05Z",
  "completed_at": "2026-08-08T08:02:30Z",
  "target_agent": "marketing-agent",
  "tool_called": "post_tweet",
  "tool_arguments": { "content": "..." },
  "hitl_required": true,
  "hitl_status": "approved",
  "hitl_approved_by": "ops.manager@edufin.co.ke",
  "hitl_approved_at": "2026-08-08T08:01:00Z",
  "result": { "tweet_id": "1823...", "posted_at": "..." },
  "error": null,
  "steps": [
    { "step": 1, "agent": "marketing-agent", "tool": "analyze_trends", "status": "completed" },
    { "step": 2, "agent": "marketing-agent", "tool": "generate_content", "status": "completed" },
    { "step": 3, "action": "hitl_review", "status": "approved" },
    { "step": 4, "agent": "marketing-agent", "tool": "post_tweet", "status": "completed" }
  ],
  "correlation_id": "corr-abc-123"
}
```

---

## 4. Human-in-the-Loop (HITL) Protocol

### 4.1 HITL Overview

The HITL protocol is the Master Agent's mechanism for **pausing autonomous execution** when a task requires human judgment. It ensures that management retains control over high-stakes, ambiguous, or potentially risky agent actions.

### 4.2 HITL Trigger Taxonomy

The Master Agent evaluates HITL requirements at three levels:

| Level | Trigger | Source | Example |
|-------|---------|--------|---------|
| **Tool-Level** | Tool annotation `hitl_required` | Sub-agent tool schema | `post_tweet` requires HITL for promotional content |
| **Task-Level** | Task classification rules | Master Agent routing logic | Any task involving financial claims requires approval |
| **Anomaly-Level** | Runtime anomaly detection | Master Agent monitoring | Sub-agent response confidence below threshold |

### 4.3 HITL Evaluation Flow

```
HITL EVALUATION FLOW
────────────────────

1. TASK RECEIVED
   │
   ▼
2. TOOL-LEVEL CHECK
   │  Query sub-agent tool schema for hitl_required annotation
   │
   ├── hitl_required = "never" ──► No HITL at tool level
   ├── hitl_required = "always" ─► HITL required (proceed to Step 4)
   └── hitl_required = "conditional" ─► Evaluate condition (Step 3)
   │
   ▼
3. CONDITIONAL EVALUATION
   │  Evaluate hitl_condition against task arguments
   │
   ├── Condition met ──► HITL required (proceed to Step 4)
   └── Condition not met ─► No HITL at tool level
   │
   ▼
4. TASK-LEVEL CHECK
   │  Evaluate task classification rules
   │
   ├── Rule match (e.g., "financial_claims") ──► HITL required
   └── No rule match ──► No HITL at task level
   │
   ▼
5. ANOMALY-LEVEL CHECK (runtime, during/after execution)
   │  Monitor sub-agent response for anomalies
   │
   ├── Confidence score < threshold ──► HITL required (post-execution review)
   ├── Unexpected output format ──► HITL required
   ├── Sub-agent self-reported uncertainty ──► HITL required
   └── No anomalies ──► No HITL at anomaly level
   │
   ▼
6. DECISION
   │
   ├── HITL required ──► Pause task, send approval request to management
   └── HITL not required ──► Execute autonomously
```

### 4.4 HITL Approval Workflow

When HITL is triggered, the Master Agent follows this workflow:

```
HITL APPROVAL WORKFLOW
──────────────────────

Master Agent                Management Interface
     │                              │
     │  1. Create HITL request      │
     │     (task_id, reason,        │
     │      proposed action,        │
     │      risk assessment)        │
     │                              │
     │  ─── notify HITL ─────────►  │
     │      (Slack + Dashboard)     │
     │                              │
     │  2. Task status =            │
     │     HITL_PENDING             │
     │     (timeout: 24 hours)      │
     │                              │
     │  ◄── approve / reject ────  │
     │      (with comment)          │
     │                              │
     ├── APPROVED                   │
     │    │                         │
     │    ▼                         │
     │  Resume execution            │
     │  Log approval                │
     │                              │
     ├── REJECTED                   │
     │    │                         │
     │    ▼                         │
     │  Cancel task                 │
     │  Log rejection               │
     │  Notify requesting system    │
     │                              │
     └── TIMEOUT (24h)              │
          │                         │
          ▼                         │
        Auto-escalate               │
        Mark as STALE               │
        Notify management (urgent)  │
```

### 4.5 HITL Request Format

```json
{
  "hitl_id": "hitl-uuid-5678",
  "task_id": "task-uuid-1234",
  "created_at": "2026-08-08T08:01:00Z",
  "expires_at": "2026-08-09T08:01:00Z",
  "trigger_level": "tool-level",
  "trigger_reason": "Tool 'post_tweet' requires HITL for content containing promotional offers",
  "proposed_action": {
    "agent": "marketing-agent",
    "tool": "post_tweet",
    "arguments": {
      "content": "🎓 New education financing packages available! Flexible repayment terms aligned with your income cycle. Apply today at edufin.co.ke #EducationFinancing #EduFin"
    }
  },
  "risk_assessment": {
    "level": "medium",
    "factors": [
      "Content contains promotional financial claims",
      "Public social media posting",
      "No regulatory disclaimers included"
    ],
    "mitigations": [
      "Content reviewed by Marketing Agent for compliance",
      "Tweet is reversible (can be deleted)",
      "No PII or financial data in content"
    ]
  },
  "approval_channels": ["slack", "dashboard", "email"],
  "status": "pending"
}
```

### 4.6 HITL Notification Channels

| Channel | Recipient | Format | Urgency |
|---------|-----------|--------|---------|
| **Slack** | `#edufin-agent-approvals` channel | Card with approve/reject buttons | Normal |
| **Web Dashboard** | Management HITL queue | Interactive approval form | Normal |
| **Email** | Management team mailing list | HTML email with approval link | Normal |
| **Slack (urgent)** | `#edufin-agent-critical` channel | @here mention | High (anomaly escalation) |

---

## 5. Anomaly Detection & Escalation

### 5.1 Anomaly Detection Mechanisms

The Master Agent continuously monitors sub-agent behavior for anomalies that warrant management attention:

| Anomaly Type | Detection Method | Threshold | Action |
|---------------|-----------------|-----------|--------|
| **Low Confidence** | Sub-agent reports confidence score | < 0.70 | HITL review before action |
| **High Uncertainty** | Sub-agent self-reports uncertainty | Explicit flag | HITL review before action |
| **Repeated Failures** | Sub-agent fails same tool > N times | N = 3 | Escalate to management |
| **Unexpected Output** | Output schema validation fails | Any mismatch | HITL review |
| **Rate Limit Hit** | External API rate limit exceeded | Any occurrence | Queue with backoff; notify if persistent |
| **Security Concern** | Tool output contains potential PII/secrets | Pattern match | Block output; HITL review |
| **Agent Unresponsive** | Sub-agent heartbeat missed | > 60 seconds | Escalate; attempt restart |
| **Cross-Agent Conflict** | Two agents produce contradictory results | Semantic analysis | HITL review |

### 5.2 Escalation Levels

```
ESCALATION HIERARCHY
────────────────────

Level 0: AUTONOMOUS
  │  Agent handles task without human involvement
  │  Logged for audit
  │
  ▼
Level 1: HITL APPROVAL
  │  Task paused; management approves/rejects
  │  Timeout: 24 hours
  │  Channel: Slack + Dashboard
  │
  ▼
Level 2: MANAGEMENT REVIEW
  │  Task failed or anomaly detected
  │  Management reviews and decides next steps
  │  Timeout: 4 hours
  │  Channel: Slack (urgent) + Dashboard + Email
  │
  ▼
Level 3: CRITICAL ESCALATION
  │  System integrity at risk (e.g., self-healing agent
  │  cannot resolve a critical infrastructure issue)
  │  Immediate management notification required
  │  Timeout: 30 minutes
  │  Channel: Slack (@here) + Phone/WhatsApp + Dashboard
  │  Action: All non-critical agent tasks paused
```

### 5.3 Escalation Notification Format

```json
{
  "escalation_id": "esc-uuid-9012",
  "task_id": "task-uuid-1234",
  "level": 2,
  "created_at": "2026-08-08T10:30:00Z",
  "reason": "Self-Healing Agent failed to resolve Laravel queue worker crash after 3 attempts",
  "agent": "self-healing-agent",
  "failed_attempts": 3,
  "last_error": "Queue worker process exits with code 137 (OOM killed) after restart",
  "context": {
    "service": "laravel-horizon",
    "server": "laravel-prod-01",
    "uptime_before_crash": "2h 14m",
    "memory_usage": "94%"
  },
  "recommended_actions": [
    "Investigate memory leak in queue worker",
    "Consider scaling up server resources",
    "Check recent deployments for regressions"
  ],
  "channels": ["slack_urgent", "dashboard", "email"],
  "status": "open"
}
```

---

## 6. State Management

### 6.1 Master Agent State

The Master Agent maintains state in Redis:

| State Key | Type | Purpose | TTL |
|-----------|------|---------|-----|
| `master:task:{task_id}` | Hash | Task record | 90 days |
| `master:hitl:{hitl_id}` | Hash | HITL request | 24 hours |
| `master:escalation:{esc_id}` | Hash | Escalation record | 30 days |
| `master:agent_status:{agent}` | String | Sub-agent health | 60 seconds |
| `master:task_queue` | List | Pending tasks | Persistent |
| `master:active_tasks` | Set | Currently executing tasks | Persistent |
| `master:metrics:daily` | Hash | Daily metrics | 7 days |

### 6.2 State Persistence

- **Task records** are persisted to Redis for fast access and to PostgreSQL (via Laravel API) for long-term audit storage
- **HITL requests** are stored in Redis with a 24-hour TTL; if approved, the full record is persisted to PostgreSQL
- **Agent health** is checked via heartbeat every 30 seconds; missed heartbeats trigger reconnection

---

## 7. Supervision & Health Monitoring

### 7.1 Sub-Agent Health Checks

| Check | Frequency | Action on Failure |
|-------|-----------|-------------------|
| **Heartbeat** | Every 30s | Mark agent as `degraded`; attempt reconnect |
| **MCP Connection** | Every 60s | Re-establish MCP session if disconnected |
| **Tool Availability** | Every 5 min | Re-query `tools/list` to detect changes |
| **Response Latency** | Continuous | If p95 > 5s, mark agent as `slow` |
| **Error Rate** | Continuous | If error rate > 10%, mark agent as `unhealthy` |

### 7.2 Agent Status States

```
┌─────────────────────────────────────────────────┐
│              AGENT STATUS STATES                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  HEALTHY ──► DEGRADED ──► UNHEALTHY ──► OFFLINE│
│     ▲           │            │            │    │
│     │           │            │            │    │
│     └───────────┴────────────┘            │    │
│     (auto-recovery)                       │    │
│                                           │    │
│                                           ▼    │
│                                      ESCALATED │
│                                      (HITL)    │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 7.3 Circuit Breaker

The Master Agent implements a circuit breaker per sub-agent to prevent cascading failures:

| State | Condition | Behavior |
|-------|-----------|----------|
| **Closed** | Normal operation | All tool calls dispatched normally |
| **Open** | > 5 consecutive failures | All tool calls immediately rejected; tasks queued |
| **Half-Open** | After 60-second cooldown | One test call allowed; if success, close circuit |

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Technical Integration & Workflow](./integration.md)
- [Marketing Agent](./marketing-agent.md)
- [Email Agent](./email-agent.md)
- [Support Agent](./support-agent.md)
- [Self-Healing Agent](./self-healing-agent.md)
