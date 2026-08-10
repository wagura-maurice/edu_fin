# EduFin Kenya — Delivery Timeframe (WordPress + Laravel)

**Version:** 1.0
**Date:** August 10, 2026
**Status:** Approved for Execution
**Author:** Delivery Planning

---

## 1. Delivery Parameters

| Parameter | Value |
|-----------|-------|
| Systems in scope | WordPress (`edufin.co.ke`) + Laravel (`app.edufin.co.ke`) |
| Out of scope (this timeframe) | AI Agents intelligence layer, mobile app build |
| Target duration | **3 months (13 weeks)** |
| Team | **Solo developer** (sequential execution) |
| Start date | **Tuesday, August 11, 2026** |
| End date | **Friday, November 7, 2026** |
| Working cadence | Mon–Fri, 5 days/week = 65 working days |
| Buffer reserved | ~5 days (Week 13) for hardening, fixes, deployment |

---

## 2. Feasibility Note (Read First)

The full scope documented in `PROJECT_DOCUMENTATION.md` §14 totals approximately **141 dev-days**:

| Track | Tasks | Estimated effort |
|-------|-------|------------------|
| Backend (BE-01…17) | 17 | 55 days |
| Frontend (FE-01…16) | 16 | 53 days |
| DevOps (DO-01…08) | 8 | 11 days |
| QA (QA-01…07) | 7 | 22+ days |
| **Total** | **48** | **~141 days** |

A solo developer has **65 working days** available. The full scope is therefore **~2.2× over capacity** and cannot be delivered in 3 months by one person.

**Resolution:** This timeframe delivers a **lean MVP** in 13 weeks. Non-critical features are explicitly deferred to **Phase 2** (post-launch, ~Week 14+). The MVP is sufficient for a **soft launch** with real users: onboarding, KYC, loan applications, admin review, and a client portal with statements. Collateral is captured as documentation only (no automated valuation). WhatsApp notifications, liability-transition automation, performance/security testing, and E2E test automation are deferred.

> If the full scope must ship in 3 months, the only viable levers are: (a) add 2–3 developers, or (b) extend to ~6 months. Both are out of scope of this plan as authorized.

---

## 3. MVP Scope Definition

### 3.1 In Scope (MVP — ships by Nov 7)

**WordPress (public site)**
- WP setup, custom theme, shared design system with Laravel
- Landing page (hero, value prop, CTAs → Laravel login/register)
- Product / financing packages pages (static content managed in wp-admin)
- About + Contact page with Contact Form 7
- Interactive loan calculator widget
- SEO basics (meta tags, sitemap) — no content marketing engine

**Laravel (core application)**
- Project scaffolding, PostgreSQL schema, migrations, seed data
- Authentication: registration, login, basic MFA (TOTP), role-based redirect
- Primary Applicant KYC onboarding wizard (multi-step + document upload to R2)
- Institution registration (basic details + fee structure)
- Education Beneficiary profiles (create, link to applicant/institution)
- Collateral module — **documentation capture only** (upload + metadata, no valuation engine)
- Loan Facility APIs + Obligor assignment (basic)
- Client portal dashboard (overview, active facilities, profile)
- Statements — generation + PDF download (basic)
- Email notifications (transactional: application received, status changes, statements)
- Admin panel via Filament (review workflow, applicant/institution/facility management)
- Core banking integration — **mock/stub layer** with defined contract (real integration in Phase 2)
- REST API (`/api/v1`) with OpenAPI documentation for mobile-app readiness
- Basic mobile responsiveness (no dedicated mobile QA)

**DevOps**
- Docker Compose dev environment, CI/CD (GitHub Actions) for Laravel
- Staging + production environments, SSL/TLS, Nginx (WP + Laravel), Cloudflare DNS/CDN
- Daily DB backups (PostgreSQL + MySQL), basic monitoring (uptime + error alerts)

**QA (lean)**
- Test plan, manual functional testing, smoke tests, one UAT round with stakeholder

### 3.2 Deferred to Phase 2 (explicitly out of MVP)

| Deferred item | Rationale | Phase 2 target |
|---------------|-----------|----------------|
| WhatsApp notifications | Email covers MVP critical paths | Weeks 14–15 |
| Liability-transition automation | Rare edge case; manual admin handling for MVP | Weeks 16–17 |
| Core banking **real** integration | Requires partner API access; mock contract delivered now | Weeks 14–18 |
| Collateral valuation engine | Documentation capture sufficient for soft launch | Weeks 18–20 |
| E2E test automation (Cypress/Playwright) | Manual + smoke testing covers MVP | Weeks 14–16 |
| Performance / load testing | Defer to pre-hardening; low early traffic expected | Week 18 |
| Security audit / penetration testing | Required before scale, not before soft launch | Week 19 |
| Blog / SEO content engine | Marketing depth, not launch-blocking | Ongoing Phase 2 |
| Full mobile responsiveness polish | Basic responsiveness in MVP; full polish later | Week 17 |
| AI Agents layer | Out of scope per authorization | Separate roadmap |
| Mobile app | API-ready; app build is a separate project | Separate roadmap |

---

## 4. Week-by-Week Delivery Plan

> Solo developer = **one workstream**. WordPress and Laravel tasks are interleaved to keep the public site and core app progressing together and to avoid context-switch overhead where dependencies exist. Each week lists the **primary deliverable** and **exit criteria**.

### Week 1 — Aug 11 → Aug 15 · Foundation & Environments
**Theme:** Stand up infrastructure and project skeletons.
- DevOps: Docker Compose dev stack (WP + Laravel + PostgreSQL + MySQL + Redis); GitHub repo + branch protection; Cloudflare DNS for `edufin.co.ke` + `app.edufin.co.ke`
- Laravel: `laravel new`, Filament installer, base config, DB connection, migration skeleton
- WordPress: WP install, starter theme scaffold, dev URL live
- **Exit:** Both apps run locally; CI pipeline green on a hello-world build.

### Week 2 — Aug 17 → Aug 22 · Data Model & Auth Backend
**Theme:** Schema and authentication core.
- Laravel: full migration set (users, applicants, institutions, beneficiaries, collateral, loan_facilities, obligors, statements, notifications, audit_log), seeders, factories
- Laravel: registration, login, logout, password reset, TOTP MFA, Sanctum tokens for API, role-based redirect middleware
- **Exit:** A user can register, verify MFA, log in, and be redirected by role. Migrations run clean on staging.

### Week 3 — Aug 24 → Aug 29 · Design System & Auth UI
**Theme:** Shared UI foundation + auth screens.
- Laravel: Blade/Livewire design system (shared CSS variables matching `IMPLEMENTATION_PLAN.md` §5.2), header/footer, layouts
- Laravel: auth UI (login, register, MFA enroll, forgot password) wired to Week 2 backend
- WordPress: theme header/footer matching shared design system, base page templates
- **Exit:** Auth flow fully usable end-to-end on Laravel; WP theme shell renders shared header.

### Week 4 — Aug 31 → Sep 5 · Public Site (WordPress)
**Theme:** Marketing site complete.
- WordPress: landing page (hero, features, CTAs → `app.edufin.co.ke/login`), product/packages pages, About + Contact (Contact Form 7), legal stubs (privacy, terms)
- WordPress: loan calculator widget (JS, static rates from wp-admin CPT)
- WordPress: SEO basics (meta, sitemap, robots), performance pass (caching, image opt)
- **Exit:** Public site presentable; calculator functional; all CTAs route to Laravel.

### Week 5 — Sep 7 → Sep 12 · Onboarding — Primary Applicant KYC (Backend)
**Theme:** Core KYC data capture backend.
- Laravel: Account Holder CRUD APIs, KYC fields, document upload endpoint (Cloudflare R2), validation rules
- Laravel: Institution management APIs (registration, fee structure)
- **Exit:** API endpoints for applicant + institution creation + document upload tested via Postman.

### Week 6 — Sep 14 → Sep 19 · Onboarding — Primary Applicant KYC (Frontend)
**Theme:** KYC wizard UI.
- Laravel: multi-step onboarding wizard (Livewire), document upload component with progress + validation
- Laravel: institution registration form, beneficiary profile forms
- **Exit:** A new applicant can complete the full KYC wizard, upload documents, and submit.

### Week 7 — Sep 21 → Sep 26 · Beneficiaries & Collateral
**Theme:** Complete the application data set.
- Laravel: Education Beneficiary APIs + UI (link to applicant/institution, age capture for liability logic)
- Laravel: Collateral module — documentation capture (upload + metadata only), linked to applicant
- **Exit:** Applicant can add beneficiaries and submit collateral documents.

### Week 8 — Sep 28 → Oct 3 · Loan Facilities & Obligor
**Theme:** Loan application creation.
- Laravel: Loan Facility APIs (create, list, detail, status), Obligor assignment APIs (basic)
- Laravel: Client dashboard — overview, active facilities, profile view
- **Exit:** A submitted application produces a loan facility record; client dashboard renders it.

### Week 9 — Oct 5 → Oct 10 · Statements & Notifications
**Theme:** Post-application client experience.
- Laravel: Statement generation service (PDF, download), statement list view in portal
- Laravel: Email notification service (transactional templates: application received, status change, statement ready), queued via Horizon
- **Exit:** Client can download a statement; status changes trigger correct emails.

### Week 10 — Oct 12 → Oct 17 · Admin Panel & Core Banking Stub
**Theme:** Staff review workflow + integration contract.
- Laravel: Filament admin — applicants, institutions, facilities, review/decision workflow, audit log view
- Laravel: Core banking integration **mock layer** with defined interface (contract documented; stub returns simulated disbursement status)
- Laravel: OpenAPI spec for `/api/v1` (mobile-app readiness)
- **Exit:** Admin can review and advance an application through statuses; banking stub responds per contract.

### Week 11 — Oct 19 → Oct 24 · Integration, API Hardening, Responsiveness
**Theme:** Wire everything together; polish.
- Laravel: end-to-end wiring review (onboarding → facility → statement → notification → admin review)
- Laravel: API auth/authorization middleware pass, rate limiting, input validation audit
- Laravel + WordPress: mobile responsiveness pass on critical flows (auth, onboarding, dashboard, public site)
- DevOps: production environment provisioned, SSL, Nginx (WP root + `/api/v1` proxy), backups verified
- **Exit:** Full happy-path works on staging from WP CTA through admin decision.

### Week 12 — Oct 26 → Oct 31 · Testing & UAT
**Theme:** Validate before launch.
- QA: execute test plan (manual functional + smoke across all MVP flows)
- QA: bug triage and fixes (priority: blockers + criticals)
- UAT: one round with stakeholder; collect feedback; fix must-fix items
- **Exit:** No open blockers/criticals; UAT sign-off on MVP scope.

### Week 13 — Nov 2 → Nov 7 · Hardening, Launch & Buffer
**Theme:** Ship it.
- Final bug fixes from UAT
- Production deployment, DNS cutover, smoke test on production
- **Soft launch** to limited beta users; monitoring active
- User/admin quick-reference docs (lean)
- Buffer for unexpected issues
- **Exit (Nov 7):** MVP live in production; Phase 2 backlog finalized.

---

## 5. Milestone Summary

| Milestone | Week ending | Deliverable |
|-----------|-------------|-------------|
| M1 — Environments live | Aug 15 | Both apps running; CI green |
| M2 — Auth + schema complete | Aug 22 | Registration/login/MFA working |
| M3 — Design system + auth UI | Aug 29 | Shared UI; auth screens usable |
| M4 — Public site complete | Sep 5 | WordPress marketing site live-ready |
| M5 — KYC onboarding complete | Sep 19 | Full applicant KYC wizard working |
| M6 — Beneficiaries + collateral | Sep 26 | Application data set complete |
| M7 — Loan facilities + dashboard | Oct 3 | Facility creation + client dashboard |
| M8 — Statements + notifications | Oct 10 | Statements + email alerts working |
| M9 — Admin panel + banking stub | Oct 17 | Review workflow + integration contract |
| M10 — Integration + responsiveness | Oct 24 | Full happy-path on staging |
| M11 — UAT sign-off | Oct 31 | No blockers; stakeholder approval |
| **M12 — Soft launch** | **Nov 7** | **MVP in production** |

---

## 6. Critical Path & Dependencies

```
Week 1  Environments
  └─► Week 2  Schema + Auth backend
        └─► Week 3  Design system + Auth UI
              ├─► Week 4  WordPress public site  (parallel-ish, low coupling)
              └─► Week 5  KYC backend
                    └─► Week 6  KYC wizard UI
                          └─► Week 7  Beneficiaries + Collateral
                                └─► Week 8  Facilities + Dashboard
                                      └─► Week 9  Statements + Notifications
                                            └─► Week 10 Admin + Banking stub
                                                  └─► Week 11 Integration + Responsive
                                                        └─► Week 12 UAT
                                                              └─► Week 13 Launch
```

**Hard dependencies (cannot be parallelized solo):**
- Auth backend (W2) → Auth UI (W3) → every downstream UI
- KYC backend (W5) → KYC UI (W6) → Beneficiaries/Collateral (W7) → Facilities (W8)
- Facilities (W8) → Statements (W9) → Admin review (W10)

**Loose coupling (can slip without blocking core):**
- WordPress public site (W4) — only dependency is shared design system (W3)
- Core banking stub (W10) — isolated behind interface

---

## 7. Risks Specific to a 3-Month Solo Delivery

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Scope creep during build | **High** | High | Frozen MVP scope (§3.1); all additions → Phase 2 backlog |
| Single-point-of-failure (one dev) | High | Critical | Daily git commits; doc decisions in repo; identify a backup contact |
| Underestimation on KYC/onboarding complexity | Medium | High | Timebox wizard to W6; if slipping, simplify validation, defer edge cases |
| Core banking partner API not ready | High | Medium | Mock contract (W10) isolates this; real integration already deferred to Phase 2 |
| MFA / Sanctum integration delays | Medium | Medium | Use Laravel Fortify + Sanctum first-party packages; avoid custom crypto |
| Document upload (R2) edge cases | Medium | Medium | Restrict to PDF/JPG/PNG; size limits; defer OCR/virus scan to Phase 2 |
| UAT feedback forces rework | Medium | High | UAT in W12 leaves W13 buffer; only must-fix items actioned |
| Burnout / illness (solo, 13 weeks) | Medium | Critical | No assumed overtime; W13 is buffer, not slack to be consumed early |

---

## 8. Effort Reconciliation (MVP vs. Capacity)

| Track | MVP effort (est.) | Notes |
|-------|-------------------|-------|
| WordPress | ~9 days | Setup, theme, landing, packages, contact, calculator |
| Laravel backend | ~30 days | Schema, auth, KYC, institutions, beneficiaries, collateral (basic), facilities, obligor, statements, email, admin, banking stub, API docs |
| Laravel frontend | ~19 days | Design system, auth UI, onboarding wizard, forms, dashboard, statements view, admin, responsiveness |
| DevOps | ~5 days | Compressed: env, CI/CD, staging/prod, SSL, backups |
| QA | ~4 days | Test plan, manual + smoke, one UAT round |
| **Total scoped** | **~67 days** | |
| Buffer (W13) | 5 days | Hardening, fixes, deployment |
| **Capacity** | **65 days** | 13 weeks × 5 |

The MVP estimate (~67 days) sits ~2 days over raw capacity. This is intentional and absorbed by: (a) the W13 buffer absorbing deployment + final fixes, and (b) solo developers typically flexing slightly longer days during a launch week. **If Week 8 milestones slip, trigger a scope cut immediately** — first candidates: defer statements PDF polish, simplify admin review to list-only, or push mobile responsiveness to Phase 2.

---

## 9. Phase 2 Outlook (post Nov 7)

Indicative only — not part of this authorized timeframe:

- **Weeks 14–15:** WhatsApp notifications, E2E test automation setup
- **Weeks 16–17:** Liability-transition automation, mobile responsiveness polish
- **Weeks 14–18:** Core banking **real** integration (partner API dependent)
- **Week 18:** Performance / load testing
- **Week 19:** Security audit + penetration testing
- **Weeks 18–20:** Collateral valuation engine
- **Ongoing:** Blog / SEO content, AI Agents layer (separate roadmap)

---

## 10. Acceptance Criteria for MVP Launch (Nov 7)

A user can, end-to-end on production:
1. Land on `edufin.co.ke`, read product info, use the loan calculator, click through to apply.
2. Register on `app.edufin.co.ke`, complete MFA enrollment.
3. Complete the KYC onboarding wizard and upload documents.
4. Register an institution and add education beneficiary(ies).
5. Submit collateral documentation.
6. Have a loan facility created and visible on the client dashboard.
7. Download a statement and receive email notifications on status changes.
8. An admin can log in to Filament, review the application, and advance its status.

Items **not** required for MVP sign-off: WhatsApp, liability-transition automation, real banking disbursement, E2E test suite, perf/security testing, mobile app.

---

## Document Control

| Field | Value |
|-------|-------|
| Created | 2026-08-10 |
| Author | Delivery Planning |
| Reviewers | (pending) |
| Next review | End of Week 4 (Sep 5) — checkpoint against milestones |
