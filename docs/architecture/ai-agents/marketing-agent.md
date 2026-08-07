# Marketing Agent

## Trend Analysis, Content Distribution, SEO & Multi-Channel Marketing Sub-Agent

**Version:** 2.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Agent Overview](#1-agent-overview)
2. [MCP Server Definition](#2-mcp-server-definition)
3. [Trend Analysis Capabilities](#3-trend-analysis-capabilities)
4. [Content Generation & Distribution](#4-content-generation--distribution)
5. [Social Media Platform Integrations](#5-social-media-platform-integrations)
6. [WordPress Content Integration](#6-wordpress-content-integration)
7. [WordPress SEO Optimization](#7-wordpress-seo-optimization)
8. [Email Marketing](#8-email-marketing)
9. [WhatsApp Marketing via WAHA](#9-whatsapp-marketing-via-waha)
10. [HITL Triggers](#10-hitl-triggers)
11. [Scheduled Workflows](#11-scheduled-workflows)

---

## 1. Agent Overview

The Marketing Agent is a specialized sub-agent responsible for **social media trend monitoring**, **content generation**, **automated content distribution** across EduFin's marketing channels, **SEO optimization** for the WordPress site, **email marketing** campaigns, and **WhatsApp marketing broadcasts**. It operates as an MCP Server, exposing tools that the Master Agent can invoke.

The agent manages marketing across all major social media platforms used in Kenya — X/Twitter, Facebook, Instagram, TikTok, LinkedIn, and YouTube — generating platform-appropriate content and distributing it subject to Human-in-the-Loop (HITL) approval where required.

### Scope Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MARKETING AGENT - SCOPE                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  IN SCOPE:                                                                      │
│  ✓ Monitor social media trends across all platforms                            │
│    (X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube)                 │
│  ✓ Generate marketing content (tweets, posts, short-form video scripts,        │
│    carousel descriptions, professional posts, blog drafts, email campaigns)    │
│  ✓ Post content to X/Twitter (@EduFinKe account)                              │
│  ✓ Post content to Facebook (EduFin Kenya page)                                │
│  ✓ Post content to Instagram (edufin.ke account)                               │
│  ✓ Post content to TikTok (edufin.ke account)                                  │
│  ✓ Post content to LinkedIn (EduFin company page)                              │
│  ✓ Post content to YouTube (EduFin channel)                                    │
│  ✓ Create draft blog posts in WordPress                                        │
│  ✓ Suggest SEO optimizations for WordPress pages/posts (drafts only)           │
│  ✓ Send marketing emails via marketing@edufin.co.ke mailbox                    │
│    (campaigns, newsletters, promotional blasts, product announcements)         │
│  ✓ Send WhatsApp marketing broadcasts via WAHA (to opted-in subscribers)       │
│  ✓ Schedule content for optimal posting times across all platforms             │
│  ✓ Analyze engagement metrics on posted content across all platforms           │
│  ✓ Monitor competitor social media activity across all platforms               │
│                                                                                 │
│  OUT OF SCOPE:                                                                  │
│  ✗ Paid advertising campaigns (Google Ads, Meta Ads, X Ads, TikTok Ads)        │
│  ✗ Modifying WordPress theme or plugin code                                    │
│  ✗ Accessing Laravel business data (loans, clients, KYC)                      │
│  ✗ Posting content without HITL approval when required                         │
│  ✗ Engaging in customer service replies on social media                        │
│  ✗ Auto-applying SEO changes to WordPress (suggestions are drafts only)        │
│  ✗ Handling info@edufin.co.ke or support@edufin.co.ke mailboxes                │
│    (handled by Email Agent)                                                     │
│  ✗ Sending WhatsApp messages to non-opted-in subscribers                       │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### External Service Dependencies

| Service | Purpose | Auth Method | MCP Client |
|---------|---------|-------------|------------|
| X/Twitter API v2 | Trend monitoring, posting | OAuth 2.0 + Bearer Token | X/Twitter MCP Server |
| Facebook Graph API | Posting to EduFin Kenya page, engagement metrics | Page Access Token (OAuth 2.0) | Facebook MCP Server |
| Instagram Graph API | Posting to edufin.ke account, engagement metrics | Instagram Graph API Token (OAuth 2.0) | Instagram MCP Server |
| TikTok Business API | Posting videos to edufin.ke account, engagement metrics | TikTok Business Access Token (OAuth 2.0) | TikTok MCP Server |
| LinkedIn Marketing API | Posting to EduFin company page, engagement metrics | OAuth 2.0 (r_organization_social + w_organization_social scopes) | LinkedIn MCP Server |
| YouTube Data API v3 | Uploading videos to EduFin channel, engagement metrics | OAuth 2.0 (youtube.upload scope) | YouTube MCP Server |
| WordPress REST API | Blog draft creation, SEO content reading | Application Password | WordPress MCP Server |
| Yoast SEO REST API | Reading SEO metadata for optimization suggestions | Application Password | WordPress MCP Server |
| Laravel REST API | Product/package data (read-only) | API Key (`X-API-Key`) | Direct HTTP |
| SMTP (marketing@edufin.co.ke) | Sending marketing emails (campaigns, newsletters) | SMTP Auth (username/password) | SMTP MCP Server |
| WAHA (WhatsApp HTTP API) | WhatsApp marketing broadcasts to opted-in subscribers | API Key (`Authorization: Bearer`) | WAHA MCP Server |

---

## 2. MCP Server Definition

### 2.1 Server Metadata

```json
{
  "serverInfo": {
    "name": "edufin-marketing-agent",
    "version": "2.0.0",
    "description": "Marketing sub-agent for trend analysis, multi-platform content distribution, SEO optimization, email marketing, and WhatsApp broadcasts",
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
| `analyze_trends` | Fetch and analyze current social media trends across all platforms | Never | No |
| `monitor_competitors` | Track competitor social media activity across all platforms | Never | No |
| `generate_content` | Generate platform-appropriate marketing content from trend/product data | Never | No |
| `generate_blog_draft` | Generate a WordPress blog post draft | Conditional | No |
| `post_tweet` | Post a tweet to @EduFinKe | Conditional | No (deletable) |
| `schedule_tweet` | Schedule a tweet for future posting | Conditional | No |
| `delete_tweet` | Delete a previously posted tweet | Always | No (reversible) |
| `post_facebook_post` | Post to the EduFin Kenya Facebook page | Conditional | No (deletable) |
| `post_instagram_post` | Post to the edufin.ke Instagram account | Conditional | No (deletable) |
| `post_tiktok_video` | Post a video to the edufin.ke TikTok account | Conditional | No (deletable) |
| `post_linkedin_post` | Post to the EduFin LinkedIn company page | Conditional | No (deletable) |
| `post_youtube_video` | Upload a video to the EduFin YouTube channel | Conditional | No (deletable) |
| `get_engagement_metrics` | Retrieve engagement metrics for posted content across all platforms | Never | No |
| `get_product_info` | Fetch product/package info from Laravel API | Never | No |
| `suggest_seo_optimization` | Analyze WordPress pages/posts and suggest SEO improvements (drafts only) | Never | No |
| `send_marketing_email` | Send a marketing email via marketing@edufin.co.ke | Conditional | No |
| `send_whatsapp_broadcast` | Send a WhatsApp marketing broadcast via WAHA to opted-in subscribers | Always | No |

### 2.3 Exposed Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://marketing/trends/current` | JSON | Current trending topics relevant to EduFin across all platforms |
| `edufin://marketing/trends/history/{days}` | JSON | Historical trend data for the past N days |
| `edufin://marketing/content/scheduled` | JSON | List of scheduled but not-yet-posted content across all platforms |
| `edufin://marketing/content/posted` | JSON | Log of posted content with engagement metrics across all platforms |
| `edufin://marketing/competitors/activity` | JSON | Recent competitor social media activity across all platforms |
| `edufin://marketing/engagement/summary` | JSON | Aggregated engagement metrics across all platforms |
| `edufin://marketing/seo/suggestions` | JSON | Current SEO optimization suggestions for WordPress pages/posts |
| `edufin://marketing/seo/audit/latest` | JSON | Latest full SEO audit report for the WordPress site |
| `edufin://marketing/email/campaigns` | JSON | Log of sent marketing email campaigns |
| `edufin://marketing/email/subscribers` | JSON | Marketing email subscriber list (count and segments, no PII exposed) |
| `edufin://marketing/whatsapp/optins` | JSON | Count of WhatsApp marketing opt-in subscribers |
| `edufin://marketing/whatsapp/broadcasts` | JSON | Log of sent WhatsApp marketing broadcasts |

### 2.4 Exposed Prompts

| Prompt Name | Arguments | Purpose |
|-------------|-----------|---------|
| `marketing_trend_report` | `period` (daily/weekly), `platform` (optional) | Generate a trend analysis report across platforms |
| `marketing_create_promotion` | `product`, `audience`, `tone`, `platform` | Create promotional content for a product on a specific platform |
| `marketing_blog_idea` | `topic`, `keywords` | Generate blog post ideas and outline |
| `marketing_seo_audit` | `scope` (page/post/all) | Generate an SEO audit report for WordPress content |
| `marketing_email_campaign` | `campaign_type`, `audience_segment`, `subject` | Generate an email marketing campaign draft |
| `marketing_whatsapp_broadcast` | `topic`, `audience_segment` | Generate a WhatsApp broadcast message draft |

---

## 3. Trend Analysis Capabilities

### 3.1 Trend Monitoring Scope

The Marketing Agent monitors trends relevant to EduFin's domain (education financing in Kenya) across **all major social media platforms used in Kenya**:

| Monitoring Target | Method | Frequency |
|-------------------|--------|-----------|
| **Hashtags (X/Twitter)** | X/Twitter API `GET /2/trends` + hashtag search | Every 2 hours |
| **Hashtags (Instagram/TikTok)** | Instagram Graph API hashtag search; TikTok trending hashtag discovery | Every 4 hours |
| **Keywords (X/Twitter)** | X/Twitter API search for: "education financing", "school fees", "student loan", "education loan Kenya", "HELB" | Every 2 hours |
| **Keywords (Facebook/LinkedIn)** | Facebook Graph API page insights & keyword search; LinkedIn content search via Marketing API | Every 4 hours |
| **Mentions (X/Twitter)** | X/Twitter API `GET /2/users/:id/mentions` for @EduFinKe | Every 30 minutes |
| **Mentions (Facebook/Instagram)** | Facebook Graph API `/{page-id}/mentions` & Instagram Graph API comments/mentions | Every 1 hour |
| **Comments (YouTube/TikTok)** | YouTube Data API `commentThreads.list`; TikTok Business API comment retrieval | Every 2 hours |
| **Competitors** | Monitor accounts/pages of competing education financing providers across all platforms | Every 4 hours |
| **Industry News** | Monitor keywords related to Kenyan education policy, CBK lending rates across X/Twitter, LinkedIn, Facebook | Daily |
| **Trending Audio/Sounds (TikTok/Instagram)** | Track trending sounds and audio clips relevant to education/finance content | Every 6 hours |
| **LinkedIn Industry Posts** | Monitor posts from Kenyan fintech and education financing thought leaders | Daily |

### 3.2 Trend Analysis Tool

**Tool: `analyze_trends`**

```json
{
  "name": "analyze_trends",
  "description": "Fetch and analyze current social media trends relevant to EduFin's education financing domain across all major platforms used in Kenya (X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube). Returns trending topics, sentiment analysis, platform-specific insights, and content recommendations.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "scope": {
        "type": "string",
        "enum": ["local", "national", "industry"],
        "description": "Geographic/topic scope of trend analysis",
        "default": "national"
      },
      "platforms": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["x_twitter", "facebook", "instagram", "tiktok", "linkedin", "youtube", "all"]
        },
        "description": "Platforms to analyze. Defaults to all.",
        "default": ["all"]
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
      "platform": "x_twitter",
      "topic": "school fees payment",
      "volume": 1250,
      "sentiment": "neutral",
      "trend_direction": "rising",
      "relevant_hashtags": ["#SchoolFees", "#EducationKenya"],
      "peak_time": "2026-08-08T09:00:00Z",
      "content_opportunity": "High engagement around school fee payment timing. Consider posting about flexible repayment schedules."
    },
    {
      "platform": "tiktok",
      "topic": "student budget hacks",
      "volume": 8900,
      "sentiment": "positive",
      "trend_direction": "rising",
      "trending_sounds": ["original_sound_12345", "audio_abcdef"],
      "content_opportunity": "Trending audio around student budgeting. Consider a short-form video on managing education finances."
    }
  ],
  "sentiment_summary": {
    "positive": 35,
    "neutral": 50,
    "negative": 15
  },
  "platform_insights": {
    "x_twitter": { "top_hashtags": ["#SchoolFees", "#HELB"], "peak_activity_hour": 12 },
    "tiktok": { "top_sounds": ["student_life_audio"], "peak_activity_hour": 19 },
    "instagram": { "top_hashtags": ["#EducationKenya", "#StudentLife"], "peak_activity_hour": 18 },
    "linkedin": { "top_topics": ["education financing", "fintech Kenya"], "peak_activity_hour": 9 }
  },
  "recommendations": [
    "Post about income-aligned repayment schedules on X/Twitter (trending topic)",
    "Create a short-form TikTok video using trending student budget audio",
    "Share a LinkedIn article on education financing trends in Kenya",
    "Avoid posting about interest rates on X/Twitter (negative sentiment trending)"
  ],
  "confidence": 0.82
}
```

### 3.3 Competitor Monitoring

**Tool: `monitor_competitors`**

The agent monitors competitor social media accounts and pages across all platforms to identify:
- Content themes and posting frequency per platform
- Engagement rates and audience growth per platform
- Promotional campaigns and offers
- Platform-specific content strategies (e.g., TikTok video styles, LinkedIn thought leadership)
- Customer complaints or sentiment shifts

Competitor accounts/pages are configured in the agent's configuration (per platform) and can be updated by management via the HITL dashboard.

---

## 4. Content Generation & Distribution

### 4.1 Content Generation Pipeline

```
CONTENT GENERATION PIPELINE
───────────────────────────

1. INPUT GATHERING
   │
   ├── Trend data (from analyze_trends, per platform)
   ├── Product info (from Laravel API: GET /api/v1/packages)
   ├── Brand guidelines (from agent config, per platform)
   ├── Posting history (avoid duplication, per platform)
   └── Platform constraints (char limits, media types, hashtag conventions)
   │
   ▼
2. CONTENT GENERATION (LLM)
   │
   ├── X/Twitter — tweet (max 280 chars) or thread
   ├── Facebook — post (text, link, image, or video)
   ├── Instagram — caption + carousel description / reel script
   ├── TikTok — short-form video script + caption (max 2200 chars)
   ├── LinkedIn — professional post (max 3000 chars) or article draft
   ├── YouTube — video title, description, tags, script outline
   ├── WordPress — blog post draft (WordPress format)
   ├── Email — campaign/newsletter/promotional blast
   └── WhatsApp — broadcast message (max 4096 chars, plain text)
   │
   ▼
3. COMPLIANCE CHECK
   │
   ├── No unsubstantiated financial claims
   ├── No PII or sensitive data
   ├── Brand voice consistency (per platform)
   ├── Regulatory disclaimers (if needed)
   ├── Content originality check
   └── Platform-specific format validation
   │
   ▼
4. HITL EVALUATION (by Master Agent)
   │
   ├── Promotional content → HITL required
   ├── Informational content → May proceed autonomously
   ├── Sensitive topics → HITL required
   ├── WhatsApp broadcasts → HITL ALWAYS required
   └── Email campaigns → HITL conditional (promotional = required)
   │
   ▼
5. DISTRIBUTION
   │
   ├── X/Twitter → post_tweet / schedule_tweet
   ├── Facebook → post_facebook_post
   ├── Instagram → post_instagram_post
   ├── TikTok → post_tiktok_video
   ├── LinkedIn → post_linkedin_post
   ├── YouTube → post_youtube_video
   ├── WordPress → generate_blog_draft (always draft, never auto-publish)
   ├── Email → send_marketing_email
   └── WhatsApp → send_whatsapp_broadcast (HITL always required)
```

### 4.2 Content Generation Tool

**Tool: `generate_content`**

```json
{
  "name": "generate_content",
  "description": "Generate platform-appropriate marketing content based on trend data and product information. Supports all major social media platforms used in Kenya. Content is returned as drafts for review — posting requires a separate tool call.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "content_type": {
        "type": "string",
        "enum": [
          "tweet",
          "thread",
          "facebook_post",
          "instagram_caption",
          "instagram_carousel",
          "instagram_reel_script",
          "tiktok_video_script",
          "linkedin_post",
          "linkedin_article",
          "youtube_video_description",
          "youtube_short_script",
          "social_post",
          "email_campaign",
          "whatsapp_broadcast"
        ],
        "description": "Type of content to generate (platform-specific)"
      },
      "topic": {
        "type": "string",
        "description": "Topic or theme for the content"
      },
      "platform": {
        "type": "string",
        "enum": ["x_twitter", "facebook", "instagram", "tiktok", "linkedin", "youtube", "email", "whatsapp"],
        "description": "Target platform for content optimization"
      },
      "product_slug": {
        "type": "string",
        "description": "Optional product slug from Laravel API for product-specific content"
      },
      "tone": {
        "type": "string",
        "enum": ["professional", "friendly", "informative", "promotional", "educational", "conversational"],
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

All generated content must adhere to EduFin brand guidelines. Platform-specific rules apply:

| Guideline | Rule |
|-----------|------|
| **Brand voice** | Professional, approachable, trustworthy |
| **Tone** | Empathetic to education financing challenges |
| **Language** | English (primary); Swahili phrases acceptable |
| **Links** | Use `edufin.co.ke` domain only |
| **Disclaimers** | Required for any content mentioning rates, terms, or financial products |
| **Prohibited** | No guarantees of loan approval; no specific interest rates without disclaimer; no competitor disparagement |

**Platform-Specific Rules:**

| Platform | Hashtag Convention | Character Limit | Media Types | Tone/Style |
|----------|-------------------|-----------------|-------------|------------|
| **X/Twitter** | Max 3 per tweet; `#EduFin` on promotional content | 280 chars (tweet); thread up to 25 tweets | Image, video (max 2:20), GIF | Concise, punchy, conversational |
| **Facebook** | Max 5 per post; `#EduFin` on promotional content | 63,206 chars (recommended < 500 for engagement) | Image, video, link, carousel, reel | Conversational, community-oriented, story-driven |
| **Instagram** | Max 30 per post (recommended 5-10); `#EduFin` always | 2,200 chars caption; 125 chars for first line | Image, carousel (max 10), reel, video (max 60s) | Visual-first, aspirational, engaging first line |
| **TikTok** | Max 100 chars of hashtags in caption; trending sounds encouraged | 2,200 chars caption (recommended < 150) | Video (3s–10min), sound, effects, stitches | Authentic, trending, youth-oriented, informal |
| **LinkedIn** | Max 5 per post; `#EduFin` on company content | 3,000 chars (post); 125,000 chars (article) | Image, video, document carousel, article | Professional, thought leadership, industry insights |
| **YouTube** | Max 15 tags; `#EduFin` in description | 100 chars title; 5,000 chars description | Video (up to 12 hours), Shorts (up to 60s) | Educational, informative, SEO-optimized titles |

---

## 5. Social Media Platform Integrations

### 5.1 X/Twitter Integration

#### 5.1.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | X/Twitter API v2 |
| Account | `@EduFinKe` |
| Auth | OAuth 2.0 (User Context) for posting; Bearer Token for reading |
| Rate Limit Strategy | Respect X API rate limits; queue if exceeded |
| Posting Limit | Max 10 tweets/day (configurable by management) |

#### 5.1.2 Post Tweet Tool

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

#### 5.1.3 X/Twitter Engagement Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Impressions | X API `GET /2/tweets/:id` | Content reach |
| Likes | X API | Audience approval |
| Retweets | X API | Content amplification |
| Replies | X API | Engagement depth |
| Link clicks | X API | Conversion intent |
| Profile clicks | X API | Interest signal |

### 5.2 Facebook Integration

#### 5.2.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | Facebook Graph API v19.0 |
| Page | EduFin Kenya page |
| Auth | Page Access Token (OAuth 2.0, requires `pages_manage_posts`, `pages_read_engagement` permissions) |
| Rate Limit Strategy | Respect Graph API rate limits; queue if exceeded |
| Posting Limit | Max 25 posts/day (configurable by management) |

#### 5.2.2 Post Facebook Post Tool

**Tool: `post_facebook_post`**

```json
{
  "name": "post_facebook_post",
  "description": "Post content to the EduFin Kenya Facebook page. Supports text, link, image, and video posts. Requires HITL approval for content containing promotional offers, financial claims, or sensitive topics.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "message": {
        "type": "string",
        "maxLength": 63206,
        "description": "Post text content (recommended under 500 chars for engagement)"
      },
      "link": {
        "type": "string",
        "format": "uri",
        "description": "Optional URL to attach (must be edufin.co.ke domain)"
      },
      "media_type": {
        "type": "string",
        "enum": ["none", "photo", "video"],
        "default": "none",
        "description": "Type of media to attach"
      },
      "media_url": {
        "type": "string",
        "format": "uri",
        "description": "URL of media to upload (required if media_type is photo or video)"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled publishing time (10 min to 75 days in advance)."
      }
    },
    "required": ["message"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains promotional offers, financial claims, rates, or product names",
    "destructive": false,
    "idempotent": false,
    "category": "social_media_posting",
    "timeout_ms": 30000,
    "retry_policy": "auto"
  }
}
```

#### 5.2.3 Facebook Engagement Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Post impressions | Graph API `/{post-id}/insights` | Content reach |
| Reactions (like, love, haha, wow, sad, angry) | Graph API | Audience sentiment |
| Comments | Graph API | Engagement depth |
| Shares | Graph API | Content amplification |
| Link clicks | Graph API | Conversion intent |
| Page follows from post | Graph API | Follower growth attribution |

### 5.3 Instagram Integration

#### 5.3.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | Instagram Graph API v19.0 |
| Account | `edufin.ke` (Business account linked to Facebook page) |
| Auth | Instagram Graph API Token (OAuth 2.0, requires `instagram_content_publish` permission) |
| Rate Limit Strategy | Respect Graph API rate limits; queue if exceeded |
| Posting Limit | Max 25 posts/day (configurable by management) |
| Media Requirement | All posts require at least one media item (image or video) |

#### 5.3.2 Post Instagram Post Tool

**Tool: `post_instagram_post`**

```json
{
  "name": "post_instagram_post",
  "description": "Post content to the edufin.ke Instagram account. Supports single image, carousel (up to 10 images), and reel/video posts. Requires HITL approval for content containing promotional offers, financial claims, or sensitive topics.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "caption": {
        "type": "string",
        "maxLength": 2200,
        "description": "Post caption (first 125 chars are most visible)"
      },
      "post_type": {
        "type": "string",
        "enum": ["single_image", "carousel", "reel", "video"],
        "description": "Type of Instagram post"
      },
      "media_urls": {
        "type": "array",
        "items": { "type": "string", "format": "uri" },
        "description": "URLs of media to publish (1 for single/reel/video, up to 10 for carousel)",
        "minItems": 1,
        "maxItems": 10
      },
      "carousel_descriptions": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Optional per-slide descriptions for carousel posts"
      },
      "location_id": {
        "type": "string",
        "description": "Optional Instagram location ID for geotagging"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled publishing time."
      }
    },
    "required": ["caption", "post_type", "media_urls"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains promotional offers, financial claims, rates, or product names",
    "destructive": false,
    "idempotent": false,
    "category": "social_media_posting",
    "timeout_ms": 60000,
    "retry_policy": "auto"
  }
}
```

#### 5.3.3 Instagram Engagement Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Impressions | Instagram Graph API `/{media-id}/insights` | Content reach |
| Reach | Instagram Graph API | Unique accounts reached |
| Likes | Instagram Graph API | Audience approval |
| Comments | Instagram Graph API | Engagement depth |
| Saves | Instagram Graph API | Content value signal |
| Profile views | Instagram Graph API | Interest signal |
| Reel plays / video views | Instagram Graph API | Video content performance |

### 5.4 TikTok Integration

#### 5.4.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | TikTok Business API (Content Posting API) |
| Account | `edufin.ke` (Business account) |
| Auth | TikTok Business Access Token (OAuth 2.0, requires `video.publish` and `video.list` scopes) |
| Rate Limit Strategy | Respect TikTok API rate limits; queue if exceeded |
| Posting Limit | Max 6 videos/day (TikTok API quota; configurable by management) |
| Video Requirements | MP4/MOV/WebM; 3s–10min duration; max 1GB file size; vertical (9:16) recommended |

#### 5.4.2 Post TikTok Video Tool

**Tool: `post_tiktok_video`**

```json
{
  "name": "post_tiktok_video",
  "description": "Post a video to the edufin.ke TikTok account. Requires a video file URL and caption. Supports trending sounds, hashtags, and privacy settings. Requires HITL approval for content containing promotional offers, financial claims, or sensitive topics.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "caption": {
        "type": "string",
        "maxLength": 2200,
        "description": "Video caption (recommended under 150 chars for engagement)"
      },
      "video_url": {
        "type": "string",
        "format": "uri",
        "description": "URL of the video file to upload (MP4/MOV/WebM, max 1GB, 3s–10min)"
      },
      "hashtags": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Hashtags to include (without # symbol, max 100 chars total)"
      },
      "sound_id": {
        "type": "string",
        "description": "Optional trending sound ID to use"
      },
      "privacy_level": {
        "type": "string",
        "enum": ["PUBLIC", "MUTUAL_FOLLOW", "FOLLOWER_OF_CREATOR", "SELF_ONLY"],
        "default": "PUBLIC",
        "description": "Video visibility setting"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled publishing time."
      }
    },
    "required": ["caption", "video_url"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains promotional offers, financial claims, rates, or product names",
    "destructive": false,
    "idempotent": false,
    "category": "social_media_posting",
    "timeout_ms": 120000,
    "retry_policy": "auto"
  }
}
```

#### 5.4.3 TikTok Engagement Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Video views | TikTok Business API `video/list` | Content reach |
| Likes | TikTok Business API | Audience approval |
| Comments | TikTok Business API | Engagement depth |
| Shares | TikTok Business API | Content amplification |
| Saves / Favorites | TikTok Business API | Content value signal |
| Average watch time | TikTok Business API | Content retention |
| Full video watch rate | TikTok Business API | Content effectiveness |

### 5.5 LinkedIn Integration

#### 5.5.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | LinkedIn Marketing API (Share API + Organization API) |
| Page | EduFin company page |
| Auth | OAuth 2.0 (requires `r_organization_social` and `w_organization_social` scopes) |
| Rate Limit Strategy | Respect LinkedIn API rate limits; queue if exceeded |
| Posting Limit | Max 10 posts/day (configurable by management) |

#### 5.5.2 Post LinkedIn Post Tool

**Tool: `post_linkedin_post`**

```json
{
  "name": "post_linkedin_post",
  "description": "Post content to the EduFin LinkedIn company page. Supports text, image, video, document carousel, and article link posts. Content should be professional and thought-leadership oriented. Requires HITL approval for content containing promotional offers, financial claims, or sensitive topics.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "text": {
        "type": "string",
        "maxLength": 3000,
        "description": "Post text content (professional tone, thought leadership)"
      },
      "media_type": {
        "type": "string",
        "enum": ["none", "image", "video", "document", "article_link"],
        "default": "none",
        "description": "Type of media to attach"
      },
      "media_url": {
        "type": "string",
        "format": "uri",
        "description": "URL of media to upload (image, video, or document PDF)"
      },
      "article_url": {
        "type": "string",
        "format": "uri",
        "description": "URL for article link post (must be edufin.co.ke domain)"
      },
      "article_title": {
        "type": "string",
        "description": "Title for article link post"
      },
      "article_description": {
        "type": "string",
        "description": "Description for article link post"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled publishing time."
      }
    },
    "required": ["text"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains promotional offers, financial claims, rates, or product names",
    "destructive": false,
    "idempotent": false,
    "category": "social_media_posting",
    "timeout_ms": 30000,
    "retry_policy": "auto"
  }
}
```

#### 5.5.3 LinkedIn Engagement Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Impressions | LinkedIn API `organizationalEntityShareStatistics` | Content reach |
| Unique impressions | LinkedIn API | Unique accounts reached |
| Likes / Reactions | LinkedIn API | Audience approval |
| Comments | LinkedIn API | Engagement depth |
| Shares | LinkedIn API | Content amplification |
| Clicks (link, CTA) | LinkedIn API | Conversion intent |
| Follows from post | LinkedIn API | Follower growth attribution |
| Engagement rate | Calculated (engagements / impressions) | Overall content effectiveness |

### 5.6 YouTube Integration

#### 5.6.1 API Configuration

| Parameter | Value |
|-----------|-------|
| API | YouTube Data API v3 |
| Channel | EduFin channel |
| Auth | OAuth 2.0 (requires `youtube.upload`, `youtube.readonly` scopes) |
| Rate Limit Strategy | Respect YouTube API quota (10,000 units/day); prioritize uploads |
| Posting Limit | Max 6 videos/day (quota-dependent; configurable by management) |
| Video Requirements | MP4, MOV, AVI, WMV, FLV; max 256GB; up to 12 hours; Shorts max 60s |

#### 5.6.2 Post YouTube Video Tool

**Tool: `post_youtube_video`**

```json
{
  "name": "post_youtube_video",
  "description": "Upload a video to the EduFin YouTube channel. Supports standard videos and Shorts. Requires a video file URL, title, description, and tags. Requires HITL approval for content containing promotional offers, financial claims, or sensitive topics.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "title": {
        "type": "string",
        "maxLength": 100,
        "description": "Video title (SEO-optimized)"
      },
      "description": {
        "type": "string",
        "maxLength": 5000,
        "description": "Video description (include links, timestamps, disclaimers)"
      },
      "video_url": {
        "type": "string",
        "format": "uri",
        "description": "URL of the video file to upload"
      },
      "tags": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Video tags for SEO (max 500 chars total, max 15 tags)"
      },
      "category_id": {
        "type": "string",
        "description": "YouTube video category ID (default: 26 - Education)",
        "default": "26"
      },
      "is_short": {
        "type": "boolean",
        "default": false,
        "description": "Whether this is a YouTube Short (vertical, max 60s)"
      },
      "privacy_status": {
        "type": "string",
        "enum": ["public", "unlisted", "private"],
        "default": "public",
        "description": "Video privacy setting"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled publishing time (requires privacy_status=private initially)."
      }
    },
    "required": ["title", "description", "video_url"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "content contains promotional offers, financial claims, rates, or product names",
    "destructive": false,
    "idempotent": false,
    "category": "social_media_posting",
    "timeout_ms": 300000,
    "retry_policy": "auto"
  }
}
```

#### 5.6.3 YouTube Engagement Metrics

| Metric | Source | Purpose |
|--------|--------|---------|
| Views | YouTube Data API `videos.list` (statistics) | Content reach |
| Likes | YouTube Data API | Audience approval |
| Comments | YouTube Data API | Engagement depth |
| Shares | YouTube Data API | Content amplification |
| Average view duration | YouTube Analytics API | Content retention |
| Click-through rate (CTR) | YouTube Analytics API | Thumbnail/title effectiveness |
| Subscribers gained | YouTube Analytics API | Channel growth attribution |
| Watch time (hours) | YouTube Analytics API | Content performance for algorithm |

### 5.7 Cross-Platform Engagement Metrics

**Tool: `get_engagement_metrics`**

Retrieves engagement data for posted content across all platforms:

```json
{
  "name": "get_engagement_metrics",
  "description": "Retrieve engagement metrics for posted content across all social media platforms (X/Twitter, Facebook, Instagram, TikTok, LinkedIn, YouTube). Supports filtering by platform and date range.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "platform": {
        "type": "string",
        "enum": ["x_twitter", "facebook", "instagram", "tiktok", "linkedin", "youtube", "all"],
        "default": "all",
        "description": "Platform to retrieve metrics for"
      },
      "content_id": {
        "type": "string",
        "description": "Optional specific content/post ID to retrieve metrics for"
      },
      "date_from": {
        "type": "string",
        "format": "date",
        "description": "Start date for metrics range"
      },
      "date_to": {
        "type": "string",
        "format": "date",
        "description": "End date for metrics range"
      }
    }
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": true,
    "category": "analytics",
    "timeout_ms": 30000
  }
}
```

The agent uses these metrics to refine future content generation — identifying which topics, tones, posting times, and content formats yield the best engagement **per platform**.

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
| `https://edufin.co.ke/wp-json/wp/v2/pages` | GET | Read pages for SEO analysis | Application Password |
| `https://edufin.co.ke/wp-json/wp/v2/media` | POST | Upload featured image | Application Password |
| `https://edufin.co.ke/wp-json/yoast/v1` | GET | Read Yoast SEO metadata for analysis | Application Password |

> **Security Note:** The Marketing Agent uses a dedicated WordPress user account with **Author** role (can create drafts, cannot publish). The Application Password is scoped to this account only. SEO suggestions are read-only recommendations — the agent does NOT modify WordPress content or Yoast SEO settings directly.

---

## 7. WordPress SEO Optimization

The Marketing Agent provides **SEO optimization suggestions** for the WordPress site. The WordPress site uses the **Yoast SEO** plugin, and the agent integrates with Yoast's REST API to read existing SEO metadata and provide improvement recommendations.

> **Important:** All SEO suggestions are **drafts/recommendations only**. The agent does NOT auto-apply any changes to WordPress content, meta descriptions, title tags, or Yoast SEO settings. Secretarial staff must review and manually apply approved suggestions.

### 7.1 SEO Optimization Tool

**Tool: `suggest_seo_optimization`**

```json
{
  "name": "suggest_seo_optimization",
  "description": "Analyze WordPress pages or posts and suggest SEO improvements. Reads content via WordPress REST API and Yoast SEO metadata via Yoast REST API. Returns draft recommendations for meta descriptions, title tags, keyword density, internal linking, image alt text, schema markup, and readability. Suggestions are NOT auto-applied — staff must review and apply manually.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "target_type": {
        "type": "string",
        "enum": ["post", "page", "all"],
        "default": "all",
        "description": "Type of content to analyze (single post, single page, or all content)"
      },
      "target_id": {
        "type": "integer",
        "description": "WordPress post/page ID (required if target_type is post or page)"
      },
      "analysis_types": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "meta_description",
            "title_tag",
            "keyword_density",
            "internal_linking",
            "image_alt_text",
            "schema_markup",
            "readability"
          ]
        },
        "description": "Specific SEO analysis types to run. Defaults to all.",
        "default": [
          "meta_description",
          "title_tag",
          "keyword_density",
          "internal_linking",
          "image_alt_text",
          "schema_markup",
          "readability"
        ]
      },
      "focus_keywords": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Optional focus keywords to evaluate against"
      }
    },
    "required": ["target_type"]
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": true,
    "category": "seo_optimization",
    "timeout_ms": 60000
  }
}
```

### 7.2 SEO Analysis Capabilities

The `suggest_seo_optimization` tool performs the following analyses:

| Analysis Type | Description | Output |
|--------------|-------------|--------|
| **Meta Description Optimization** | Evaluates existing Yoast meta description for length (120–155 chars), keyword inclusion, and click-worthiness. Suggests improved meta descriptions. | Draft meta description suggestions |
| **Title Tag Optimization** | Analyzes title tags for length (50–60 chars), keyword placement, and click-through appeal. Suggests optimized title tags. | Draft title tag suggestions |
| **Content Keyword Density** | Analyzes body content for focus keyword density (recommended 0.5%–2.5%), keyword variations, and semantic keywords (LSI). Flags over-optimization. | Keyword density report with recommendations |
| **Internal Linking Suggestions** | Scans content for internal linking opportunities to other EduFin pages/posts. Identifies orphaned content and suggests relevant internal links. | List of suggested internal links with anchor text |
| **Image Alt Text Suggestions** | Identifies images missing alt text or with poor alt text. Suggests descriptive, keyword-relevant alt text for each image. | Draft alt text for each image |
| **Schema Markup Recommendations** | Evaluates whether appropriate schema markup (Article, FAQPage, BreadcrumbList, Organization) is present. Recommends schema types to add via Yoast. | Schema markup recommendations |
| **Readability Analysis** | Analyzes content readability (Flesch Reading Ease, sentence length, paragraph length, transition words, passive voice). Aligns with Yoast readability checks. | Readability score and improvement suggestions |

### 7.3 Yoast SEO Integration

The Marketing Agent integrates with the Yoast SEO plugin via its REST API to read existing SEO metadata:

| Yoast API Endpoint | Method | Purpose |
|---------------------|--------|---------|
| `https://edufin.co.ke/wp-json/yoast/v1/get_head` | GET | Retrieve Yoast SEO head data (meta description, title, OG tags) for a post/page |
| `https://edufin.co.ke/wp-json/wp/v2/posts/{id}?_embed` | GET | Retrieve full post content with Yoast meta fields (`yoast_head`, `yoast_head_json`) |

The agent reads the following Yoast SEO fields for analysis:
- `yoast_head_json.title` — Current SEO title
- `yoast_head_json.description` — Current meta description
- `yoast_head_json.og_title` — Open Graph title
- `yoast_head_json.og_description` — Open Graph description
- `yoast_head_json.schema` — Current schema markup

> **Note:** The agent only **reads** Yoast SEO data. It does NOT write to Yoast SEO fields. All optimization suggestions are returned as draft recommendations for staff to review and apply manually via the WordPress admin interface.

### 7.4 SEO Suggestion Response Structure

```json
{
  "target": {
    "type": "post",
    "id": 142,
    "url": "https://edufin.co.ke/blog/education-financing-options-kenya",
    "title": "Education Financing Options in Kenya"
  },
  "analyses": {
    "meta_description": {
      "current": "Learn about education financing options in Kenya.",
      "issues": ["Length is 48 chars (recommended 120-155)", "Missing focus keyword in first half"],
      "suggestions": [
        "Discover the best education financing options in Kenya, including student loans, HELB, and EduFin's flexible repayment plans. Compare and apply today."
      ]
    },
    "title_tag": {
      "current": "Education Financing Options in Kenya | EduFin",
      "issues": ["Length is 48 chars (within range)", "Could include current year for freshness"],
      "suggestions": [
        "Education Financing Options in Kenya 2026 | EduFin"
      ]
    },
    "keyword_density": {
      "focus_keyword": "education financing Kenya",
      "current_density": 0.3,
      "recommended_range": "0.5% - 2.5%",
      "issues": ["Keyword density is below recommended range"],
      "suggestions": ["Include the focus keyword 3-4 more times naturally in the content body"]
    },
    "internal_linking": {
      "orphaned_content": false,
      "suggested_links": [
        { "anchor_text": "student loan repayment plans", "target_url": "https://edufin.co.ke/our-packages" },
        { "anchor_text": "HELB application guide", "target_url": "https://edufin.co.ke/blog/helb-application-guide" }
      ]
    },
    "image_alt_text": {
      "images_missing_alt": 2,
      "suggestions": [
        { "image_url": "https://edufin.co.ke/wp-content/uploads/2026/07/students.jpg", "suggested_alt": "Kenyan students reviewing education financing options" }
      ]
    },
    "schema_markup": {
      "current_schema": ["Organization", "BreadcrumbList"],
      "missing_schema": ["Article", "FAQPage"],
      "suggestions": ["Add Article schema for this blog post", "Consider adding FAQPage schema if FAQ section is present"]
    },
    "readability": {
      "flesch_reading_ease": 48.5,
      "rating": "difficult",
      "issues": ["Average sentence length is 24 words (recommended < 20)", "23% passive voice (recommended < 10%)"],
      "suggestions": ["Shorten sentences", "Reduce passive voice usage", "Add more transition words"]
    }
  },
  "overall_seo_score": 62,
  "priority_fixes": [
    "Improve meta description length (high impact)",
    "Increase keyword density (medium impact)",
    "Add missing alt text to 2 images (medium impact)"
  ]
}
```

### 7.5 SEO Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://marketing/seo/suggestions` | JSON | Current SEO optimization suggestions for WordPress pages/posts |
| `edufin://marketing/seo/audit/latest` | JSON | Latest full SEO audit report for the WordPress site |

---

## 8. Email Marketing

The Marketing Agent handles outbound marketing email communications via the dedicated `marketing@edufin.co.ke` mailbox. This **overlaps** with the Email Agent's domain — the Email Agent handles `info@edufin.co.ke` and `support@edufin.co.ke` for general and support email, while the Marketing Agent handles `marketing@edufin.co.ke` specifically for outbound marketing campaigns.

### 8.1 Mailbox Configuration

| Parameter | Value |
|-----------|-------|
| Mailbox | `marketing@edufin.co.ke` |
| Purpose | Outbound marketing emails (campaigns, newsletters, promotional blasts, product announcements) |
| SMTP Server | `smtp.edufin.co.ke` (or provider SMTP) |
| SMTP Auth | Username/password (stored in agent secrets) |
| Encryption | TLS (STARTTLS port 587 or SSL port 465) |
| Sending Limit | Configurable by management (default: 500 emails/day) |
| Managed By | Marketing Agent (this agent) |

> **Note on Email Agent overlap:** The Email Agent continues to handle `info@edufin.co.ke` and `support@edufin.co.ke` for general inquiries and customer support. The Marketing Agent's email capability is **limited to the `marketing@` mailbox** for outbound marketing campaigns only. The Marketing Agent does NOT read or respond to inbound emails — it sends outbound marketing content only.

### 8.2 Send Marketing Email Tool

**Tool: `send_marketing_email`**

```json
{
  "name": "send_marketing_email",
  "description": "Send a marketing email via the marketing@edufin.co.ke mailbox. Supports newsletters, promotional blasts, and product announcements. Recipients are drawn from the marketing subscriber list (opt-in only). Requires HITL approval for promotional content.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "campaign_type": {
        "type": "string",
        "enum": ["newsletter", "promotional_blast", "product_announcement", "educational_content"],
        "description": "Type of marketing email campaign"
      },
      "subject": {
        "type": "string",
        "maxLength": 100,
        "description": "Email subject line"
      },
      "preheader": {
        "type": "string",
        "maxLength": 100,
        "description": "Email preheader/preview text"
      },
      "body_html": {
        "type": "string",
        "description": "HTML body content of the email"
      },
      "body_text": {
        "type": "string",
        "description": "Plain text fallback body content"
      },
      "audience_segment": {
        "type": "string",
        "description": "Subscriber segment to send to (e.g., 'all', 'newsletter_subscribers', 'product_updates')",
        "default": "all"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled sending time. If omitted, sends immediately."
      }
    },
    "required": ["campaign_type", "subject", "body_html"]
  },
  "annotations": {
    "hitl_required": "conditional",
    "hitl_condition": "campaign_type is promotional_blast or product_announcement, or content contains financial claims/rates",
    "destructive": false,
    "idempotent": false,
    "category": "email_marketing",
    "timeout_ms": 60000,
    "retry_policy": "auto"
  }
}
```

### 8.3 Email Campaign Types

| Campaign Type | Description | HITL Required | Frequency Guidance |
|---------------|-------------|---------------|-------------------|
| **Newsletter** | Regular newsletter with educational content, company updates, and blog highlights | Conditional (if promotional content included) | Bi-weekly |
| **Promotional Blast** | Time-sensitive promotional content (e.g., back-to-school financing offer) | Always | Max 2/month |
| **Product Announcement** | New product/package launch announcement | Always | As needed |
| **Educational Content** | Educational content series (e.g., "Understanding education loans") | Conditional (if product mentions included) | Weekly |

### 8.4 Email Marketing Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://marketing/email/campaigns` | JSON | Log of sent marketing email campaigns |
| `edufin://marketing/email/subscribers` | JSON | Marketing email subscriber list (count and segments, no PII exposed) |

---

## 9. WhatsApp Marketing via WAHA

The Marketing Agent can send WhatsApp marketing broadcasts via **WAHA (WhatsApp HTTP API)**, a self-hosted REST API server for WhatsApp message sending and receiving. WhatsApp marketing messages are sent only to subscribers who have **explicitly opted in** to WhatsApp marketing communications.

### 9.1 WAHA Server Configuration

| Parameter | Value |
|-----------|-------|
| Service | WAHA (WhatsApp HTTP API) |
| Base URL | `https://waha.edufin.co.ke` (self-hosted) |
| Auth | API Key via `Authorization: Bearer {API_KEY}` header |
| API Version | WAHA REST API (v1) |
| Session | `edufin-marketing` (dedicated WhatsApp session) |
| Webhook URL | `https://agents.edufin.co.ke/webhooks/waha/delivery` (for delivery reports) |
| Webhook Events | `message.sent`, `message.delivered`, `message.read`, `message.failed` |
| Rate Limit | Respect WAHA rate limits; queue if exceeded |
| Sending Limit | Max 100 broadcasts/day (configurable by management) |
| Opt-In Requirement | Only subscribers who have explicitly opted in to WhatsApp marketing |

### 9.2 WAHA API Endpoints Used

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `POST /api/v1/messages/send-text` | POST | Send a text message to a recipient |
| `POST /api/v1/messages/send-image` | POST | Send an image message with caption |
| `POST /api/v1/sessions/start` | POST | Start the WhatsApp session (if not active) |
| `GET /api/v1/sessions/me` | GET | Check session status |
| `POST /api/v1/webhook` | POST | Register/update webhook for delivery reports |

### 9.3 Send WhatsApp Broadcast Tool

**Tool: `send_whatsapp_broadcast`**

```json
{
  "name": "send_whatsapp_broadcast",
  "description": "Send a WhatsApp marketing broadcast via WAHA to opted-in subscribers. Only sends to subscribers who have explicitly opted in to WhatsApp marketing. ALWAYS requires HITL approval — no autonomous sending. Supports text and image messages.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "message_type": {
        "type": "string",
        "enum": ["text", "image"],
        "default": "text",
        "description": "Type of WhatsApp message to send"
      },
      "message": {
        "type": "string",
        "maxLength": 4096,
        "description": "Message text content (plain text, max 4096 chars)"
      },
      "image_url": {
        "type": "string",
        "format": "uri",
        "description": "URL of image to send (required if message_type is image)"
      },
      "image_caption": {
        "type": "string",
        "maxLength": 4096,
        "description": "Caption for image message"
      },
      "audience_segment": {
        "type": "string",
        "description": "Opt-in subscriber segment to send to (e.g., 'all_opted_in', 'whatsapp_promo')",
        "default": "all_opted_in"
      },
      "scheduled_at": {
        "type": "string",
        "format": "date-time",
        "description": "Optional scheduled sending time. If omitted, sends immediately after HITL approval."
      }
    },
    "required": ["message_type", "message"]
  },
  "annotations": {
    "hitl_required": "always",
    "hitl_condition": "All WhatsApp marketing broadcasts require explicit human approval before sending",
    "destructive": false,
    "idempotent": false,
    "category": "whatsapp_marketing",
    "timeout_ms": 120000,
    "retry_policy": "auto"
  }
}
```

> **HITL Note:** WhatsApp marketing broadcasts **always** require HITL approval, regardless of content type. This is a stricter policy than other platforms because WhatsApp has stricter anti-spam policies and violations can result in account bans. Only subscribers who have explicitly opted in to WhatsApp marketing receive broadcasts.

### 9.4 WhatsApp Marketing Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://marketing/whatsapp/optins` | JSON | Count of WhatsApp marketing opt-in subscribers |
| `edufin://marketing/whatsapp/broadcasts` | JSON | Log of sent WhatsApp marketing broadcasts with delivery status |

### 9.5 WAHA Webhook: Delivery Reports

WAHA sends delivery report webhooks to the Marketing Agent's webhook endpoint. The agent processes the following events:

| Event | Description | Agent Action |
|-------|-------------|--------------|
| `message.sent` | Message was sent to WhatsApp server | Update broadcast log status to "sent" |
| `message.delivered` | Message was delivered to recipient's device | Update broadcast log status to "delivered" |
| `message.read` | Message was read by recipient | Update broadcast log status to "read" |
| `message.failed` | Message failed to send | Update broadcast log status to "failed"; alert management via HITL dashboard |

---

## 10. HITL Triggers

### 10.1 Marketing-Specific HITL Conditions

| Condition | Trigger | Rationale |
|-----------|---------|-----------|
| Promotional content | Any post mentions a product, offer, or financing package (any platform) | Financial promotions require compliance review |
| Financial claims | Content mentions interest rates, approval likelihood, or terms (any platform) | Regulatory compliance (CBK guidelines) |
| Sensitive topics | Content references economic hardship, default, or debt (any platform) | Brand sensitivity; avoid appearing exploitative |
| High posting frequency | > 5 posts on a single platform in a single day | Prevent spam-like behavior |
| New competitor mention | Content references a competitor by name (any platform) | Legal/brand risk |
| Trending negative sentiment | Industry sentiment is trending negative on any platform | Avoid appearing tone-deaf |
| Blog draft with claims | Blog post contains specific financial claims | Requires compliance review before staff sees it |
| WhatsApp broadcast | Any WhatsApp marketing broadcast | Always required — WhatsApp anti-spam policy risk |
| Email promotional blast | Email campaign_type is promotional_blast or product_announcement | Promotional emails require compliance review |
| SEO suggestions with claims | SEO meta description suggestions contain financial claims | Ensure compliance before staff applies |
| YouTube video upload | Video content contains promotional or financial claims | Video content is permanent and high-visibility |

### 10.2 Always-Autonomous Actions (No HITL)

| Action | Rationale |
|--------|-----------|
| `analyze_trends` | Read-only; no public-facing output |
| `monitor_competitors` | Read-only; no public-facing output |
| `get_engagement_metrics` | Read-only; no public-facing output |
| `get_product_info` | Read-only; internal data fetch |
| `generate_content` (informational) | Draft only; no public posting |
| `suggest_seo_optimization` | Read-only analysis; suggestions are drafts, not auto-applied |

### 10.3 Always-HITL Actions

| Action | Rationale |
|--------|-----------|
| `send_whatsapp_broadcast` | WhatsApp anti-spam policy risk; always requires human approval |
| `delete_tweet` | Deletion is a destructive action; always requires confirmation |

---

## 11. Scheduled Workflows

### 11.1 Default Schedule

| Workflow | Schedule | Description |
|----------|----------|-------------|
| Daily trend scan | 08:00 EAT daily | Run `analyze_trends` across all platforms for the past 24 hours |
| Competitor check | 10:00 EAT daily | Run `monitor_competitors` across all platforms |
| Content generation | 09:00 EAT daily | Generate 1-3 content variants per platform based on trend data |
| Engagement report | 17:00 EAT Friday | Weekly engagement metrics summary across all platforms |
| Blog idea generation | 10:00 EAT Monday | Generate 2-3 blog post ideas for the week |
| SEO audit | 09:00 EAT Monday | Weekly full SEO audit of WordPress site via `suggest_seo_optimization` |
| SEO suggestions review | 14:00 EAT Wednesday | Mid-week SEO suggestion refresh for recently published content |
| Newsletter campaign | 10:00 EAT every other Monday | Generate and prepare bi-weekly newsletter for HITL review |
| WhatsApp broadcast prep | 11:00 EAT Thursday | Generate WhatsApp broadcast draft for weekly HITL review |

### 11.2 Posting Schedule

The Marketing Agent suggests optimal posting times based on historical engagement data **per platform**:

| Platform | Time Slot (EAT) | Day | Engagement Rationale |
|----------|------------------|-----|----------------------|
| **X/Twitter** | 08:00 - 09:00 | Mon-Fri | Morning commute; high mobile usage |
| **X/Twitter** | 12:00 - 13:00 | Mon-Fri | Lunch break; peak engagement |
| **X/Twitter** | 17:00 - 18:00 | Mon-Thu | Evening commute |
| **Facebook** | 09:00 - 10:00 | Mon-Fri | Morning browsing; high reach |
| **Facebook** | 13:00 - 14:00 | Mon-Fri | Lunch break; peak engagement |
| **Facebook** | 15:00 - 16:00 | Sat-Sun | Weekend afternoon browsing |
| **Instagram** | 11:00 - 13:00 | Mon-Fri | Midday browsing; high engagement |
| **Instagram** | 18:00 - 20:00 | Mon-Fri | Evening; peak Instagram usage |
| **Instagram** | 10:00 - 12:00 | Saturday | Weekend morning browsing |
| **TikTok** | 12:00 - 13:00 | Mon-Fri | Lunch break; high mobile usage |
| **TikTok** | 19:00 - 21:00 | Mon-Sun | Evening; peak TikTok usage |
| **TikTok** | 10:00 - 12:00 | Saturday | Weekend morning browsing |
| **LinkedIn** | 08:00 - 10:00 | Tue-Thu | Morning; professional audience active |
| **LinkedIn** | 12:00 - 13:00 | Tue-Thu | Lunch break; professional browsing |
| **YouTube** | 14:00 - 16:00 | Thu-Sun | Afternoon; higher watch time |
| **YouTube Shorts** | 18:00 - 20:00 | Mon-Fri | Evening; short-form content peak |
| **Email (newsletter)** | 09:00 - 10:00 | Tuesday | Morning; high open rates |
| **Email (promo)** | 10:00 - 11:00 | Wed-Thu | Midweek; higher engagement |
| **WhatsApp** | 12:00 - 13:00 | Thu-Fri | Lunch break; high read rates |

> All scheduled posts still go through HITL evaluation. Scheduled content that requires approval is sent to management 2 hours before the scheduled posting time. WhatsApp broadcasts require HITL approval regardless of schedule.

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Master Agent Architecture](./master-agent.md)
- [Support Agent](./support-agent.md)
- [WordPress Architecture](../wordpress/README.md)
- [Technical Integration & Workflow](./integration.md)
