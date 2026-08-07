# Marketing Agent

## Trend Analysis & Content Distribution Sub-Agent

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Agent Overview](#1-agent-overview)
2. [MCP Server Definition](#2-mcp-server-definition)
3. [Trend Analysis Capabilities](#3-trend-analysis-capabilities)
4. [Content Generation & Distribution](#4-content-generation--distribution)
5. [X/Twitter Integration](#5-xtwitter-integration)
6. [WordPress Content Integration](#6-wordpress-content-integration)
7. [HITL Triggers](#7-hitl-triggers)
8. [Scheduled Workflows](#8-scheduled-workflows)

---

## 1. Agent Overview

The Marketing Agent is a specialized sub-agent responsible for **social media trend monitoring**, **content generation**, and **automated content distribution** across EduFin's marketing channels. It operates as an MCP Server, exposing tools that the Master Agent can invoke.

### Scope Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MARKETING AGENT - SCOPE                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  IN SCOPE:                                                                      │
│  ✓ Monitor social media trends (X/Twitter, relevant hashtags)                  │
│  ✓ Generate marketing content (tweets, blog drafts, social posts)              │
│  ✓ Post content to X/Twitter (@EduFinKe account)                              │
│  ✓ Create draft blog posts in WordPress                                        │
│  ✓ Schedule content for optimal posting times                                  │
│  ✓ Analyze engagement metrics on posted content                                │
│  ✓ Monitor competitor social media activity                                    │
│                                                                                 │
│  OUT OF SCOPE:                                                                  │
│  ✗ Direct email marketing (handled by Email Agent)                             │
│  ✗ Paid advertising campaigns (Google Ads, X Ads)                             │
│  ✗ Modifying WordPress theme or plugin code                                    │
│  ✗ Accessing Laravel business data (loans, clients, KYC)                      │
│  ✗ Posting content without HITL approval when required                         │
│  ✗ Engaging in customer service replies on social media                        │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### External Service Dependencies

| Service | Purpose | Auth Method | MCP Client |
|---------|---------|-------------|------------|
| X/Twitter API v2 | Trend monitoring, posting | OAuth 2.0 + Bearer Token | X/Twitter MCP Server |
| WordPress REST API | Blog draft creation | Application Password | WordPress MCP Server |
| Laravel REST API | Product/package data (read-only) | API Key (`X-API-Key`) | Direct HTTP |

---

## 2. MCP Server Definition

### 2.1 Server Metadata

```json
{
  "serverInfo": {
    "name": "edufin-marketing-agent",
    "version": "1.0.0",
    "description": "Marketing sub-agent for trend analysis and content distribution",
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
| `analyze_trends` | Fetch and analyze current social media trends | Never | No |
| `monitor_competitors` | Track competitor social media activity | Never | No |
| `generate_content` | Generate marketing content from trend/product data | Never | No |
| `generate_blog_draft` | Generate a WordPress blog post draft | Conditional | No |
| `post_tweet` | Post a tweet to @EduFinKe | Conditional | No (deletable) |
| `schedule_tweet` | Schedule a tweet for future posting | Conditional | No |
| `delete_tweet` | Delete a previously posted tweet | Always | No (reversible) |
| `get_engagement_metrics` | Retrieve engagement metrics for posted content | Never | No |
| `get_product_info` | Fetch product/package info from Laravel API | Never | No |

### 2.3 Exposed Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://marketing/trends/current` | JSON | Current trending topics relevant to EduFin |
| `edufin://marketing/trends/history/{days}` | JSON | Historical trend data for the past N days |
| `edufin://marketing/content/scheduled` | JSON | List of scheduled but not-yet-posted content |
| `edufin://marketing/content/posted` | JSON | Log of posted content with engagement metrics |
| `edufin://marketing/competitors/activity` | JSON | Recent competitor social media activity |
| `edufin://marketing/engagement/summary` | JSON | Aggregated engagement metrics |

### 2.4 Exposed Prompts

| Prompt Name | Arguments | Purpose |
|-------------|-----------|---------|
| `marketing_trend_report` | `period` (daily/weekly) | Generate a trend analysis report |
| `marketing_create_promotion` | `product`, `audience`, `tone` | Create promotional content for a product |
| `marketing_blog_idea` | `topic`, `keywords` | Generate blog post ideas and outline |

---

## 3. Trend Analysis Capabilities

### 3.1 Trend Monitoring Scope

The Marketing Agent monitors trends relevant to EduFin's domain (education financing in Kenya):

| Monitoring Target | Method | Frequency |
|-------------------|--------|-----------|
| **Hashtags** | X/Twitter API `GET /2/trends` + hashtag search | Every 2 hours |
| **Keywords** | X/Twitter API search for: "education financing", "school fees", "student loan", "education loan Kenya", "HELB" | Every 2 hours |
| **Mentions** | X/Twitter API `GET /2/users/:id/mentions` for @EduFinKe | Every 30 minutes |
| **Competitors** | Monitor accounts of competing education financing providers | Every 4 hours |
| **Industry News** | Monitor keywords related to Kenyan education policy, CBK lending rates | Daily |

### 3.2 Trend Analysis Tool

**Tool: `analyze_trends`**

```json
{
  "name": "analyze_trends",
  "description": "Fetch and analyze current social media trends relevant to EduFin's education financing domain. Returns trending topics, sentiment analysis, and content recommendations.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "scope": {
        "type": "string",
        "enum": ["local", "national", "industry"],
        "description": "Geographic/topic scope of trend analysis",
        "default": "national"
      },
      "keywords": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Additional keywords to monitor"
      },
      "period_hours": {
        "type": "integer",
        "description": "Look-back period in hours",
        "default": 24,
        "minimum": 1,
        "maximum": 168
      }
    }
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": true,
    "category": "trend_analysis",
    "timeout_ms": 30000
  }
}
```

**Response Structure:**
```json
{
  "trends": [
    {
      "topic": "school fees payment",
      "volume": 1250,
      "sentiment": "neutral",
      "trend_direction": "rising",
      "relevant_hashtags": ["#SchoolFees", "#EducationKenya"],
      "peak_time": "2026-08-08T09:00:00Z",
      "content_opportunity": "High engagement around school fee payment timing. Consider posting about flexible repayment schedules."
    }
  ],
  "sentiment_summary": {
    "positive": 35,
    "neutral": 50,
    "negative": 15
  },
  "recommendations": [
    "Post about income-aligned repayment schedules (trending topic)",
    "Avoid posting about interest rates (negative sentiment trending)"
  ],
  "confidence": 0.82
}
```

### 3.3 Competitor Monitoring

**Tool: `monitor_competitors`**

The agent monitors competitor social media accounts to identify:
- Content themes and posting frequency
- Engagement rates and audience growth
- Promotional campaigns and offers
- Customer complaints or sentiment shifts

Competitor accounts are configured in the agent's configuration and can be updated by management via the HITL dashboard.

---

## 4. Content Generation & Distribution

### 4.1 Content Generation Pipeline

```
CONTENT GENERATION PIPELINE
───────────────────────────

1. INPUT GATHERING
   │
   ├── Trend data (from analyze_trends)
   ├── Product info (from Laravel API: GET /api/v1/packages)
   ├── Brand guidelines (from agent config)
   └── Posting history (avoid duplication)
   │
   ▼
2. CONTENT GENERATION (LLM)
   │
   ├── Draft tweet(s) — max 280 chars
   ├── Draft blog post — WordPress format
   ├── Hashtag selection
   └── Optimal posting time suggestion
   │
   ▼
3. COMPLIANCE CHECK
   │
   ├── No unsubstantiated financial claims
   ├── No PII or sensitive data
   ├── Brand voice consistency
   ├── Regulatory disclaimers (if needed)
   └── Content originality check
   │
   ▼
4. HITL EVALUATION (by Master Agent)
   │
   ├── Promotional content → HITL required
   ├── Informational content → May proceed autonomously
   └── Sensitive topics → HITL required
   │
   ▼
5. DISTRIBUTION
   │
   ├── X/Twitter → post_tweet / schedule_tweet
   └── WordPress → generate_blog_draft (always draft, never auto-publish)
```

### 4.2 Content Generation Tool

**Tool: `generate_content`**

```json
{
  "name": "generate_content",
  "description": "Generate marketing content (tweets, social posts) based on trend data and product information. Content is returned as drafts for review — posting requires a separate tool call.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "content_type": {
        "type": "string",
        "enum": ["tweet", "thread", "social_post"],
        "description": "Type of content to generate"
      },
      "topic": {
        "type": "string",
        "description": "Topic or theme for the content"
      },
      "product_slug": {
        "type": "string",
        "description": "Optional product slug from Laravel API for product-specific content"
      },
      "tone": {
        "type": "string",
        "enum": ["professional", "friendly", "informative", "promotional"],
        "default": "professional"
      },
      "max_variants": {
        "type": "integer",
        "description": "Number of content variants to generate",
        "default": 3,
        "minimum": 1,
        "maximum": 5
      }
    },
    "required": ["content_type", "topic"]
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": false,
    "category": "content_generation",
    "timeout_ms": 60000
  }
}
```

### 4.3 Brand Guidelines

All generated content must adhere to EduFin brand guidelines:

| Guideline | Rule |
|-----------|------|
| **Brand voice** | Professional, approachable, trustworthy |
| **Tone** | Empathetic to education financing challenges |
| **Language** | English (primary); Swahili phrases acceptable |
| **Hashtags** | Max 3 per tweet; include `#EduFin` on promotional content |
| **Links** | Use `edufin.co.ke` domain only |
| **Disclaimers** | Required for any content mentioning rates, terms, or financial products |
| **Prohibited** | No guarantees of loan approval; no specific interest rates without disclaimer; no competitor disparagement |

---

## 5. X/Twitter Integration

### 5.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | X/Twitter API v2 |
| Account | `@EduFinKe` |
| Auth | OAuth 2.0 (User Context) for posting; Bearer Token for reading |
| Rate Limit Strategy | Respect X API rate limits; queue if exceeded |
| Posting Limit | Max 10 tweets/day (configurable by management) |

### 5.2 Post Tweet Tool

**Tool: `post_tweet`**

```json
{
  "name": "post_tweet",
  "description": "Post a tweet to the @EduFinKe X/Twitter account. Requires HITL approval for content containing promotional offers, financial claims, or sensitive topics.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "content": {
        "type": "string",
        "maxLength": 280,
        "description": "Tweet text content"
      },
      "media_ids": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Optional X media IDs for attached images"
      },
      "reply_to_tweet_id": {
        "type": "string",
        "description": "Optional tweet ID to reply to"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled posting time. If omitted, posts immediately."
      }
    },
    "required": ["content"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains promotional offers, financial claims, rates, or product names",
    "destructive": false,
    "idempotent": false,
    "category": "social_media_posting",
    "timeout_ms": 15000,
    "retry_policy": "auto"
  }
}
```

### 5.3 Engagement Metrics

**Tool: `get_engagement_metrics`**

Retrieves engagement data for posted tweets:

| Metric | Source | Purpose |
|--------|--------|---------|
| Impressions | X API `GET /2/tweets/:id` | Content reach |
| Likes | X API | Audience approval |
| Retweets | X API | Content amplification |
| Replies | X API | Engagement depth |
| Link clicks | X API | Conversion intent |
| Profile clicks | X API | Interest signal |

The agent uses these metrics to refine future content generation — identifying which topics, tones, and posting times yield the best engagement.

---

## 6. WordPress Content Integration

### 6.1 Blog Draft Creation

The Marketing Agent can create **draft** blog posts in WordPress via the WordPress REST API. Drafts are never auto-published — they require secretarial staff review.

**Tool: `generate_blog_draft`**

```json
{
  "name": "generate_blog_draft",
  "description": "Generate a blog post draft and save it to WordPress as a draft for secretarial staff review. The draft is NOT published automatically.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "title": {
        "type": "string",
        "description": "Blog post title"
      },
      "topic": {
        "type": "string",
        "description": "Topic or theme for the blog post"
      },
      "keywords": {
        "type": "array",
        "items": { "type": "string" },
        "description": "SEO keywords to target"
      },
      "word_count": {
        "type": "integer",
        "description": "Target word count",
        "default": 800,
        "minimum": 300,
        "maximum": 2000
      }
    },
    "required": ["title", "topic"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains financial claims or product-specific information",
    "destructive": false,
    "idempotent": false,
    "category": "content_creation",
    "timeout_ms": 120000
  }
}
```

### 6.2 WordPress REST API Integration

| API Endpoint | Method | Purpose | Auth |
|--------------|--------|---------|------|
| `https://edufin.co.ke/wp-json/wp/v2/posts` | POST | Create draft blog post | Application Password |
| `https://edufin.co.ke/wp-json/wp/v2/posts` | GET | List existing posts (avoid duplication) | Application Password |
| `https://edufin.co.ke/wp-json/wp/v2/media` | POST | Upload featured image | Application Password |

> **Security Note:** The Marketing Agent uses a dedicated WordPress user account with **Author** role (can create drafts, cannot publish). The Application Password is scoped to this account only.

---

## 7. HITL Triggers

### 7.1 Marketing-Specific HITL Conditions

| Condition | Trigger | Rationale |
|-----------|---------|-----------|
| Promotional content | Tweet mentions a product, offer, or financing package | Financial promotions require compliance review |
| Financial claims | Content mentions interest rates, approval likelihood, or terms | Regulatory compliance (CBK guidelines) |
| Sensitive topics | Content references economic hardship, default, or debt | Brand sensitivity; avoid appearing exploitative |
| High posting frequency | > 5 tweets in a single day | Prevent spam-like behavior |
| New competitor mention | Content references a competitor by name | Legal/brand risk |
| Trending negative sentiment | Industry sentiment is trending negative | Avoid appearing tone-deaf |
| Blog draft with claims | Blog post contains specific financial claims | Requires compliance review before staff sees it |

### 7.2 Always-Autonomous Actions (No HITL)

| Action | Rationale |
|--------|-----------|
| `analyze_trends` | Read-only; no public-facing output |
| `monitor_competitors` | Read-only; no public-facing output |
| `get_engagement_metrics` | Read-only; no public-facing output |
| `get_product_info` | Read-only; internal data fetch |
| `generate_content` (informational) | Draft only; no public posting |

---

## 8. Scheduled Workflows

### 8.1 Default Schedule

| Workflow | Schedule | Description |
|----------|----------|-------------|
| Daily trend scan | 08:00 EAT daily | Run `analyze_trends` for the past 24 hours |
| Competitor check | 10:00 EAT daily | Run `monitor_competitors` |
| Content generation | 09:00 EAT daily | Generate 1-3 tweet variants based on trend data |
| Engagement report | 17:00 EAT Friday | Weekly engagement metrics summary |
| Blog idea generation | 10:00 EAT Monday | Generate 2-3 blog post ideas for the week |

### 8.2 Posting Schedule

The Marketing Agent suggests optimal posting times based on historical engagement data:

| Time Slot (EAT) | Day | Engagement Rationale |
|------------------|-----|----------------------|
| 08:00 - 09:00 | Mon-Fri | Morning commute; high mobile usage |
| 12:00 - 13:00 | Mon-Fri | Lunch break; peak engagement |
| 17:00 - 18:00 | Mon-Thu | Evening commute |
| 10:00 - 11:00 | Saturday | Weekend morning browsing |

> All scheduled posts still go through HITL evaluation. Scheduled content that requires approval is sent to management 2 hours before the scheduled posting time.

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Master Agent Architecture](./master-agent.md)
- [WordPress Architecture](../wordpress/README.md)
- [Technical Integration & Workflow](./integration.md)
