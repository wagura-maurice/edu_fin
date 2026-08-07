# Self-Healing Agent

## Infrastructure Monitoring & Code Repair Sub-Agent

**Version:** 1.0  
**Last Updated:** August 8, 2026

---

## Table of Contents

1. [Agent Overview](#1-agent-overview)
2. [MCP Server Definition](#2-mcp-server-definition)
3. [Monitoring Scope](#3-monitoring-scope)
4. [WordPress Environment Healing](#4-wordpress-environment-healing)
5. [Laravel Environment Healing](#5-laravel-environment-healing)
6. [Code Repair Pipeline](#6-code-repair-pipeline)
7. [Deployment & Rollback Safety](#7-deployment--rollback-safety)
8. [HITL Triggers](#8-hitl-triggers)
9. [Scheduled Workflows](#9-scheduled-workflows)

---

## 1. Agent Overview

The Self-Healing Agent is a specialized sub-agent responsible for **monitoring the health** of EduFin's WordPress and Laravel environments and **autonomously repairing** common issues in the core source code and infrastructure. It operates as an MCP Server, exposing diagnostic and repair tools to the Master Agent.

### Scope Definition

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    SELF-HEALING AGENT - SCOPE                                  │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  IN SCOPE:                                                                      │
│  ✓ Monitor WordPress server health (PHP-FPM, Nginx, MySQL, Redis)             │
│  ✓ Monitor Laravel server health (PHP-FPM, Nginx, PostgreSQL, Redis, Horizon) │
│  ✓ Detect and diagnose application errors (PHP errors, exceptions, crashes)    │
│  ✓ Monitor error logs and identify recurring patterns                          │
│  ✓ Generate code patches for identified bugs                                   │
│  ✓ Apply patches to non-production environments autonomously                   │
│  ✓ Restart failed services (queue workers, PHP-FPM, etc.)                     │
│  ✓ Clear caches and optimize database tables                                   │
│  ✓ Create Git branches with proposed fixes                                     │
│  ✓ Generate pull request descriptions for human review                         │
│                                                                                 │
│  OUT OF SCOPE:                                                                  │
│  ✗ Deploying patches directly to production without HITL approval              │
│  ✗ Modifying database schema or running migrations autonomously                │
│  ✗ Modifying security configurations (firewall, SSL, RBAC)                    │
│  ✗ Modifying core banking integration code                                     │
│  ✗ Handling hardware failures (disk, memory, network) — escalate              │
│  ✗ Modifying DNS or Cloudflare configurations                                  │
│  ✗ Accessing or modifying PII or financial data                                │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Safety Principles

| Principle | Implementation |
|-----------|----------------|
| **Do No Harm** | All production changes require HITL approval |
| **Reversibility** | Every patch must have a documented rollback procedure |
| **Isolation** | Patches are tested in staging before production consideration |
| **Minimal Change** | Patches address only the identified issue; no scope creep |
| **Full Traceability** | Every change is committed to Git with issue reference and agent attribution |
| **Production Protection** | Core banking and financial transaction code is never auto-modified |

### External Service Dependencies

| Service | Purpose | Auth Method |
|---------|---------|-------------|
| Git Repository | Code access, branch creation, commits | SSH deploy key |
| WordPress Server (SSH) | Log access, service restarts, health checks | SSH key |
| Laravel Server (SSH) | Log access, service restarts, health checks | SSH key |
| Laravel API Health | `GET /api/v1/health` endpoint | API Key |
| Cloudflare API | Edge health, cache purge | API Token (read-only + cache purge) |

---

## 2. MCP Server Definition

### 2.1 Server Metadata

```json
{
  "serverInfo": {
    "name": "edufin-selfhealing-agent",
    "version": "1.0.0",
    "description": "Self-healing sub-agent for monitoring and repairing WordPress and Laravel environments",
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
| `run_diagnostic` | Run full diagnostic on a target system | Never | No |
| `check_health` | Quick health check for a specific service | Never | No |
| `read_logs` | Read and analyze error logs | Never | No |
| `identify_issue` | Analyze logs/errors to identify root cause | Never | No |
| `generate_patch` | Generate a code patch for an identified issue | Never | No |
| `apply_patch_staging` | Apply a patch to the staging environment | Conditional | Yes (reversible) |
| `apply_patch_production` | Apply a patch to production | Always | Yes (reversible) |
| `restart_service` | Restart a failed service | Conditional | Yes (reversible) |
| `clear_cache` | Clear application or server cache | Never | No |
| `optimize_db` | Run database optimization (non-destructive) | Conditional | No |
| `create_pr` | Create a Git branch and pull request with a fix | Never | No |
| `rollback_deployment` | Rollback a recent deployment | Always | Yes (reversible) |

### 2.3 Exposed Resources

| Resource URI | Type | Description |
|--------------|------|-------------|
| `edufin://selfhealing/health/wordpress` | JSON | WordPress server health status |
| `edufin://selfhealing/health/laravel` | JSON | Laravel server health status |
| `edufin://selfhealing/logs/{service}` | JSON | Recent error logs for a service |
| `edufin://selfhealing/issues/active` | JSON | Currently identified but unresolved issues |
| `edufin://selfhealing/issues/history` | JSON | Historical log of detected and resolved issues |
| `edufin://selfhealing/patches/proposed` | JSON | Generated patches awaiting review |
| `edufin://selfhealing/patches/applied` | JSON | Log of applied patches with Git references |

### 2.4 Exposed Prompts

| Prompt Name | Arguments | Purpose |
|-------------|-----------|---------|
| `selfheal_diagnose` | `target` (wordpress/laravel), `symptom` | Run diagnostic for a specific symptom |
| `selfheal_patch` | `issue_id`, `target_file` | Generate a patch for a specific issue |
| `selfheal_root_cause` | `log_entries`, `error_type` | Perform root cause analysis from logs |

---

## 3. Monitoring Scope

### 3.1 WordPress Monitoring

| Component | Check | Frequency | Alert Threshold |
|-----------|-------|-----------|-----------------|
| **PHP-FPM** | Process alive, response time | 60s | Down or > 5s response |
| **Nginx** | HTTP 200 on `edufin.co.ke` | 60s | Non-200 status |
| **MySQL** | Connection, slow query count | 120s | Connection failure or > 10 slow queries/min |
| **Redis** | Connection, memory usage | 60s | Connection failure or > 90% memory |
| **WordPress** | `wp-login.php` reachable | 300s | Non-200 status |
| **Error Logs** | PHP error log scan | 120s | > 10 errors in 5 minutes |
| **Disk Space** | Disk usage percentage | 300s | > 85% disk usage |
| **CPU/Memory** | System resource usage | 60s | > 90% sustained for 5 min |
| **SSL Certificate** | Certificate expiry | 1 hour | < 14 days to expiry |

### 3.2 Laravel Monitoring

| Component | Check | Frequency | Alert Threshold |
|-----------|-------|-----------|-----------------|
| **PHP-FPM** | Process alive, response time | 60s | Down or > 5s response |
| **Nginx** | HTTP 200 on `app.edufin.co.ke` | 60s | Non-200 status |
| **PostgreSQL** | Connection, slow query count | 120s | Connection failure or > 10 slow queries/min |
| **Redis** | Connection, memory usage | 60s | Connection failure or > 90% memory |
| **Horizon** | Queue worker status, job backlog | 60s | Workers down or > 1000 pending jobs |
| **API Health** | `GET /api/v1/health` endpoint | 60s | Non-200 or degraded status |
| **Error Logs** | Laravel log scan | 120s | > 10 errors in 5 minutes |
| **Disk Space** | Disk usage percentage | 300s | > 85% disk usage |
| **CPU/Memory** | System resource usage | 60s | > 90% sustained for 5 min |
| **Queue Workers** | Horizon worker process count | 60s | Worker count < configured minimum |
| **Failed Jobs** | Failed job count in Horizon | 120s | > 50 failed jobs in 1 hour |
| **SSL Certificate** | Certificate expiry | 1 hour | < 14 days to expiry |

### 3.3 Health Check Tool

**Tool: `check_health`**

```json
{
  "name": "check_health",
  "description": "Perform a quick health check on a specific service or component in the WordPress or Laravel environment.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "target": {
        "type": "string",
        "enum": ["wordpress", "laravel"],
        "description": "Which platform to check"
      },
      "component": {
        "type": "string",
        "enum": ["php_fpm", "nginx", "database", "redis", "horizon", "api", "disk", "cpu_memory", "ssl"],
        "description": "Specific component to check"
      }
    },
    "required": ["target", "component"]
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": true,
    "category": "monitoring",
    "timeout_ms": 15000
  }
}
```

**Response Example:**
```json
{
  "target": "laravel",
  "component": "horizon",
  "status": "degraded",
  "details": {
    "workers_expected": 3,
    "workers_active": 1,
    "pending_jobs": 847,
    "failed_jobs_last_hour": 23,
    "last_heartbeat": "2026-08-08T10:29:30Z"
  },
  "recommendation": "2 Horizon workers are not responding. Consider restarting the Horizon service.",
  "confidence": 0.95
}
```

---

## 4. WordPress Environment Healing

### 4.1 WordPress Autonomous Repairs (No HITL Required)

| Issue | Repair Action | Reversible |
|-------|--------------|------------|
| Cache bloat | Flush Redis object cache | Yes |
| Transient overflow | Clean expired transients | Yes |
| High PHP-FPM memory | Restart PHP-FPM pool (if configured) | Yes |
| Stale page cache | Purge WP Rocket cache | Yes |
| Broken permalink | Flush rewrite rules | Yes |

### 4.2 WordPress Repairs Requiring HITL

| Issue | Repair Action | HITL Level |
|-------|--------------|------------|
| Plugin conflict | Disable conflicting plugin | Level 1 |
| Theme error | Switch to default theme temporarily | Level 1 |
| Database corruption | Run `wp db repair` | Level 2 |
| Core file modification | Restore original WordPress core file | Level 2 |
| White screen of death | Disable all plugins, re-enable one by one | Level 2 |

### 4.3 WordPress Code Repair Scope

The Self-Healing Agent can generate patches for:

| Code Area | Repair Capability | HITL for Apply |
|-----------|------------------|----------------|
| Custom theme (`wp-content/themes/edufin/`) | PHP errors, template bugs, missing functions | Yes (production) |
| `wp-config.php` | Configuration errors (never security settings) | Yes |
| `.htaccess` | Rewrite rule issues | Yes |
| Plugin compatibility | Compatibility patches for approved plugins | Yes |
| WordPress core files | Restore from official source (never modify) | Yes |

> **Hard Rule:** The Self-Healing Agent will **never** modify WordPress core files. If a core file is corrupted, the agent restores it from the official WordPress release, verified by checksum.

---

## 5. Laravel Environment Healing

### 5.1 Laravel Autonomous Repairs (No HITL Required)

| Issue | Repair Action | Reversible |
|-------|--------------|------------|
| Cache bloat | `php artisan cache:clear` | Yes |
| Config cache stale | `php artisan config:cache` | Yes |
| Route cache stale | `php artisan route:cache` | Yes |
| View cache stale | `php artisan view:clear` | Yes |
| Stale Horizon stats | `php artisan horizon:purge` | Yes |
| Failed jobs backlog | Retry failed jobs (`php artisan queue:retry`) | Yes |

### 5.2 Laravel Repairs Requiring HITL

| Issue | Repair Action | HITL Level |
|-------|--------------|------------|
| Horizon workers crashed | Restart Horizon service | Level 1 |
| Queue worker OOM | Restart with increased memory limit | Level 1 |
| PostgreSQL connection pool exhausted | Restart PgBouncer (if used) | Level 2 |
| Laravel application down | `php artisan up` (after diagnosis) | Level 2 |
| Migration needed | **Never autonomous** — escalate to Level 2 | Level 2 |

### 5.3 Laravel Code Repair Scope

The Self-Healing Agent can generate patches for:

| Code Area | Repair Capability | HITL for Apply |
|-----------|------------------|----------------|
| `app/Services/` | Bug fixes in business service classes | Yes (production) |
| `app/Http/Controllers/` | Controller bug fixes | Yes (production) |
| `app/Jobs/` | Queue job bug fixes | Yes (production) |
| `app/Models/` | Model relationship or accessor fixes | Yes (production) |
| `routes/` | Route definition fixes | Yes (production) |
| `config/` | Configuration fixes (never security config) | Yes (production) |
| `composer.json` | Dependency version fixes | Yes (production) |

### 5.4 Code Areas NEVER Modified by Agent

| Code Area | Reason |
|-----------|--------|
| `app/Services/Banking/` | Core banking integration — too critical for autonomous repair |
| `app/Services/Notification/` | Notification dispatch — requires careful review |
| Database migrations (`database/migrations/`) | Schema changes are irreversible; always manual |
| Security middleware | Security-critical; requires security team review |
| Authentication code (`app/Http/Controllers/Auth/`) | Security-critical |
| `config/auth.php`, `config/sanctum.php` | Security configuration |
| `.env` file | Environment secrets; never modified by agent |

> **Hard Rule:** Issues detected in the "never modified" code areas are escalated to Level 2 (Management Review) with a diagnostic report and recommended fix, but the agent does not generate or apply patches autonomously.

---

## 6. Code Repair Pipeline

### 6.1 Issue Detection to Patch Application Flow

```
CODE REPAIR PIPELINE
────────────────────

1. ISSUE DETECTED
   │  (monitoring alert, log analysis, health check failure)
   │
   ▼
2. DIAGNOSTIC & ROOT CAUSE ANALYSIS
   │  (run_diagnostic, read_logs, identify_issue)
   │
   ├── Issue in "never modify" zone ──► ESCALATE (Level 2)
   ├── Hardware failure ──► ESCALATE (Level 3)
   ├── Service crash (restartable) ──► restart_service (HITL Level 1)
   └── Code bug ──► Continue to Step 3
   │
   ▼
3. PATCH GENERATION
   │  (generate_patch)
   │  • Read source file from Git
   │  • Analyze the bug with LLM
   │  • Generate minimal diff
   │  • Include tests (if test infrastructure exists)
   │
   ▼
4. PATCH VALIDATION
   │  • Syntax check (PHP -l)
   │  • Static analysis (if configured: PHPStan, Psalm)
   │  • Run relevant tests (if test suite exists)
   │  • Verify diff is minimal and scoped
   │
   ├── Validation failed ──► Revise patch (max 3 attempts) ──► Escalate if still failing
   └── Validation passed ──► Continue to Step 5
   │
   ▼
5. STAGING APPLICATION
   │  (apply_patch_staging — HITL Level 1)
   │  • Apply patch to staging branch
   │  • Deploy to staging environment
   │  • Run smoke tests
   │  • Monitor for 30 minutes
   │
   ├── Staging tests failed ──► Rollback staging, revise patch
   └── Staging tests passed ──► Continue to Step 6
   │
   ▼
6. PRODUCTION DEPLOYMENT (HITL Level 2)
   │  (apply_patch_production — Always HITL)
   │  • Create PR with patch description
   │  • Request management approval
   │  • On approval: merge to main, deploy
   │  • Monitor production for 1 hour post-deploy
   │
   ├── Production issue post-deploy ──► rollback_deployment (HITL Level 3)
   └── Production stable ──► Mark issue as RESOLVED
```

### 6.2 Generate Patch Tool

**Tool: `generate_patch`**

```json
{
  "name": "generate_patch",
  "description": "Generate a code patch for an identified issue. The patch is returned as a unified diff with metadata. The patch is NOT applied — application requires a separate tool call with HITL approval.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "issue_id": {
        "type": "string",
        "description": "Identifier of the diagnosed issue"
      },
      "target_platform": {
        "type": "string",
        "enum": ["wordpress", "laravel"],
        "description": "Which platform the patch targets"
      },
      "target_file": {
        "type": "string",
        "description": "Path to the file to patch (relative to project root)"
      },
      "issue_description": {
        "type": "string",
        "description": "Description of the issue being fixed"
      },
      "root_cause": {
        "type": "string",
        "description": "Root cause analysis from identify_issue"
      }
    },
    "required": ["issue_id", "target_platform", "target_file", "issue_description"]
  },
  "annotations": {
    "hitl_required": "never",
    "destructive": false,
    "idempotent": false,
    "category": "code_repair",
    "timeout_ms": 120000
  }
}
```

**Response Example:**
```json
{
  "patch_id": "patch-uuid-1234",
  "issue_id": "issue-uuid-5678",
  "target_platform": "laravel",
  "target_file": "app/Services/Loan/LoanCalculatorService.php",
  "diff": "--- a/app/Services/Loan/LoanCalculatorService.php\n+++ b/app/Services/Loan/LoanCalculatorService.php\n@@ -45,7 +45,7 @@\n-    return $principal * $rate * $tenure;\n+    return $principal * ($rate / 12) * $tenure;\n",
  "description": "Fix: Monthly interest rate was calculated using annual rate instead of monthly rate (annual / 12). This caused monthly payment calculations to be 12x higher than expected.",
  "validation": {
    "syntax_check": "passed",
    "static_analysis": "passed",
    "test_results": "5 passed, 0 failed"
  },
  "rollback_plan": "Revert commit via git revert. No database changes involved.",
  "risk_level": "medium",
  "confidence": 0.88
}
```

### 6.3 Patch Metadata Requirements

Every generated patch includes:

| Field | Description |
|-------|-------------|
| `patch_id` | Unique identifier for the patch |
| `issue_id` | Link to the diagnosed issue |
| `target_file` | File being modified |
| `diff` | Unified diff format |
| `description` | Human-readable description of the fix |
| `root_cause` | Root cause analysis |
| `validation` | Syntax, static analysis, and test results |
| `rollback_plan` | Steps to revert the patch |
| `risk_level` | `low`, `medium`, `high` (determined by code criticality) |
| `confidence` | Agent confidence in the fix (0.0 - 1.0) |

---

## 7. Deployment & Rollback Safety

### 7.1 Deployment Safety Rules

| Rule | Enforcement |
|------|-------------|
| No direct production edits | Patches are applied via Git PR → merge → deploy pipeline |
| Staging validation required | All patches must pass staging validation before production HITL request |
| Monitoring post-deploy | Agent monitors the affected service for 1 hour after production deployment |
| Auto-rollback on regression | If post-deploy health check fails, agent initiates rollback (HITL Level 3) |
| One patch at a time | No parallel patches to the same service to isolate impact |
| Git attribution | All commits include `Co-Authored-By: Self-Healing Agent <agent@edufin.co.ke>` |

### 7.2 Rollback Procedure

```
ROLLBACK PROCEDURE
──────────────────

1. POST-DEPLOY ISSUE DETECTED
   │  (health check failure, error spike, test failure)
   │
   ▼
2. ASSESS SEVERITY
   │
   ├── Minor (error rate < 1%) ──► Monitor; may self-resolve
   ├── Moderate (error rate 1-5%) ──► Notify management; recommend rollback
   └── Severe (error rate > 5% or service down) ──► Initiate emergency rollback
   │
   ▼
3. EMERGENCY ROLLBACK (Severe only)
   │  • git revert <patch_commit>
   │  • Deploy previous version
   │  • Verify health restoration
   │  • Notify management (Level 3 escalation)
   │  • Create incident report
   │
   ▼
4. POST-ROLLBACK ANALYSIS
   │  • Analyze why patch failed in production
   │  • Update patch validation criteria
   │  • Log incident for future reference
```

### 7.3 Git Workflow

```
Git Branch Strategy for Agent Patches
─────────────────────────────────────

main (production)
  │
  ├── staging (staging environment)
  │     │
  │     └── agent/fix-{issue_id}  ← Agent creates this branch
  │           │
  │           └── Patch applied here
  │                 │
  │                 ├── Validation passed → Merge to staging
  │                 │                       │
  │                 │                       └── HITL approved → Merge to main
  │                 │
  │                 └── Validation failed → Branch discarded
  │
  └── (existing development branches unaffected)
```

---

## 8. HITL Triggers

### 8.1 Self-Healing-Specific HITL Conditions

| Condition | Trigger | HITL Level |
|-----------|---------|------------|
| Restart service | `restart_service` called for any service | Level 1 |
| Apply patch to staging | `apply_patch_staging` | Level 1 |
| Apply patch to production | `apply_patch_production` | Level 2 (always) |
| Rollback deployment | `rollback_deployment` | Level 2 (or Level 3 for emergency) |
| Database optimization | `optimize_db` on production database | Level 1 |
| Plugin disable (WordPress) | Disabling a plugin to diagnose conflict | Level 1 |
| Theme switch (WordPress) | Temporarily switching to default theme | Level 1 |
| Issue in "never modify" zone | Issue detected in banking/auth/security code | Level 2 |
| Hardware failure detected | Disk failure, memory failure, network outage | Level 3 |
| Multiple services failing | > 2 services down simultaneously | Level 3 |
| Patch confidence < 0.70 | Agent's confidence in generated patch is low | Level 2 |
| Recurring issue | Same issue detected > 3 times in 7 days | Level 2 |

### 8.2 Always-Autonomous Actions (No HITL)

| Action | Rationale |
|--------|-----------|
| `run_diagnostic` | Read-only; no system changes |
| `check_health` | Read-only; no system changes |
| `read_logs` | Read-only; no system changes |
| `identify_issue` | Analysis only; no system changes |
| `generate_patch` | Generates a diff; does not apply anything |
| `create_pr` | Creates a PR for review; does not merge or deploy |
| `clear_cache` | Non-destructive; standard maintenance |
| Cache flush (Redis) | Non-destructive; standard maintenance |

---

## 9. Scheduled Workflows

### 9.1 Default Monitoring Schedule

| Workflow | Schedule | Description |
|----------|----------|-------------|
| WordPress health check | Every 60s | Check all WordPress components |
| Laravel health check | Every 60s | Check all Laravel components |
| Log scan (WordPress) | Every 120s | Scan PHP error logs for new errors |
| Log scan (Laravel) | Every 120s | Scan Laravel logs for new errors |
| Disk space check | Every 5 min | Check disk usage on both servers |
| SSL certificate check | Every 1 hour | Check certificate expiry |
| Full diagnostic | 06:00 EAT daily | Comprehensive diagnostic on both platforms |
| Failed job review | Every 30 min | Review Laravel Horizon failed jobs |
| Stale issue cleanup | 18:00 EAT daily | Mark issues unresolved > 7 days as escalated |

### 9.2 Maintenance Windows

The Self-Healing Agent respects maintenance windows during which autonomous repairs are paused:

| Window | Time (EAT) | Purpose |
|--------|------------|---------|
| Daily maintenance | 02:00 - 03:00 | System backups; agent pauses non-critical repairs |
| Sunday maintenance | 02:00 - 05:00 | Weekly deep maintenance; agent monitoring only |

> During maintenance windows, the agent continues **monitoring** but does not attempt **repairs** unless a Level 3 (critical) issue is detected.

---

**See Also:**
- [AI Agents Overview](./README.md)
- [MCP Protocol Specification](./mcp-protocol.md)
- [Master Agent Architecture](./master-agent.md)
- [Marketing Agent](./marketing-agent.md)
- [Email Agent](./email-agent.md)
- [Support Agent](./support-agent.md)
- [Technical Integration & Workflow](./integration.md)
- [WordPress Architecture](../wordpress/README.md)
- [Laravel Architecture](../laravel/README.md)
- [Deployment Documentation](../../deployment/README.md)
