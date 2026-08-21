# Public Loan Tracking

**Version:** 1.0
**Last Updated:** August 19, 2026
**Status:** Specification (Laravel backend pending implementation)

---

## 1. Technical Overview

### 1.1 Purpose

The Public Loan Tracking feature allows anyone — applicants, obligors, or third parties — to check the status of a loan application by entering a tracking number on the WordPress website, **without logging in to the Laravel client portal**.

This addresses a current UX gap: the footer "Track Application" link points to `app.edufin.co.ke/dashboard`, which requires authentication. Users who only want a quick status check are forced through a login flow. The public tracking page provides a frictionless alternative while keeping all sensitive data behind the authenticated portal.

### 1.2 Architecture

The feature follows the same dual-platform pattern already established by the client onboarding wizard:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                      PUBLIC LOAN TRACKING FLOW                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  WORDPRESS (edufin.co.ke/track-application)     LARAVEL API                 │
│  ───────────────────────────────────────────    ──────────────────────────  │
│                                                                              │
│  1. User visits the "Track Application" page                                 │
│     on the WordPress site.                                                   │
│                                                                              │
│  2. User enters their tracking number                                        │
│     (e.g. EDF-2026-08X7K2)                                                   │
│     and passes Turnstile verification.                                       │
│                                                                              │
│  3. JavaScript sends ──────────────────────────►  GET /api/v1/loans/track/   │
│     the request with                                {reference}              │
│     X-API-Key header                                X-API-Key: {key}         │
│     ◄───────────────────────────────────────────  Returns whitelisted        │
│     JSON response                                  public fields only        │
│                                                                              │
│  4. JavaScript renders                                                        │
│     the status card:                                                          │
│     - Status badge (PENDING/ACTIVE/etc.)                                     │
│     - Package name                                                            │
│     - Beneficiary first name + last initial                                  │
│     - Application date                                                        │
│     - Last updated date                                                       │
│     - "View full details" link → app.edufin.co.ke/login                      │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

| Layer | Responsibility |
|-------|---------------|
| **WordPress (UI)** | Page template, form, Turnstile widget, JavaScript API call, result rendering. No server-side data storage. |
| **Laravel (API)** | Lookup by `facility_reference`, field whitelisting via `LoanTrackingResource`, rate limiting, 404 for not-found. |
| **Cloudflare** | Turnstile bot protection on the form, WAF rules, CDN caching of the page HTML. |

### 1.3 Key Design Decisions

1. **No authentication required** — The tracking lookup is anonymous. The reference number itself is the "credential." This is intentional: the feature is for quick status checks, not for accessing account details.

2. **Field whitelisting at the API layer** — Laravel returns only explicitly approved public fields. The full Loan Facility model is never serialized. This ensures that even if WordPress is compromised, no sensitive data can be leaked through this endpoint.

3. **Reference numbers are unguessable** — The `facility_reference` format (`EDF-YYYY-XXXXXX` where XXXXXX is 6 random alphanumeric characters) makes enumeration impractical. Combined with rate limiting and Turnstile, the risk of brute-force discovery is negligible.

4. **404 does not reveal existence** — Whether the reference is not found, is malformed, or belongs to a deleted record, the API returns the same 404 response. This prevents information enumeration.

5. **Mock-first development** — During the current phase (Laravel not yet built), WordPress serves mock REST endpoints that return the same JSON shape. Switching to production is a config change, identical to the onboarding wizard migration path.

6. **No server-side proxy** — The API call is made client-side from the browser, not server-side from WordPress. This keeps WordPress stateless and avoids adding Laravel-coupling to WordPress's PHP execution. The `X-API-Key` is already public (used by the onboarding wizard's `/options/*` endpoints).

---

## 2. Laravel Side — API Specification

### 2.1 Endpoint

```
GET /api/v1/loans/track/{reference}
```

**Authentication:** Public API key (`X-API-Key` header). No JWT required.

**Rate Limit:** 20 requests per minute per IP address (stricter than the default 100 req/min public limit, to discourage enumeration).

### 2.2 Request

**Headers:**

| Header | Required | Description |
|--------|----------|-------------|
| `X-API-Key` | Yes | Public API key |
| `Accept` | No | `application/json` (default) |

**URL Parameter:**

| Parameter | Type | Format | Description |
|-----------|------|--------|-------------|
| `reference` | String | `EDF-YYYY-XXXXXX` | The `facility_reference` to look up. Validated server-side with regex `^EDF-\d{4}-[A-Z0-9]{6}$`. |

**Example Request:**

```http
GET /api/v1/loans/track/EDF-2026-08X7K2 HTTP/1.1
Host: edufin.co.ke
X-API-Key: {public_api_key}
Accept: application/json
```

### 2.3 Response — Success (200 OK)

```json
{
  "success": true,
  "data": {
    "reference": "EDF-2026-08X7K2",
    "status": "ACTIVE",
    "status_label": "Active",
    "package_name": "University Financing",
    "beneficiary_initials": "John D.",
    "application_date": "2026-07-15",
    "last_updated": "2026-08-01",
    "portal_url": "https://app.edufin.co.ke/login"
  }
}
```

**Field Definitions:**

| Field | Type | Source | Description |
|-------|------|--------|-------------|
| `reference` | String | `facility_reference` | The tracking number, echoed back for confirmation |
| `status` | Enum | `LoanFacility.status` | One of: `PENDING`, `ACTIVE`, `PAID`, `DEFAULTED`, `RESTRUCTURED` |
| `status_label` | String | Derived | Human-readable label (e.g., "Pending Review", "Active", "Fully Paid") |
| `package_name` | String | Related `FinancingPackage.name` | The financing package name (e.g., "University Financing") |
| `beneficiary_initials` | String | `Beneficiary.first_name` + first letter of `last_name` + "." | Partial name only — no full surname, no contact details |
| `application_date` | Date (Y-m-d) | `LoanFacility.created_at` | Date the application was submitted |
| `last_updated` | Date (Y-m-d) | Most recent status change timestamp | Date of the last status transition |
| `portal_url` | String | Config constant | URL to the authenticated portal login, for users who want full details |

### 2.4 Response — Not Found (404)

```json
{
  "success": false,
  "message": "No application found with that tracking number. Please check and try again."
}
```

> **Security:** The 404 response is identical whether the reference does not exist, is malformed, or belongs to a deleted/archived record. The API never confirms whether a reference exists — it only returns data for valid, active records.

### 2.5 Response — Rate Limited (429)

```json
{
  "success": false,
  "message": "Too many requests. Please try again in a minute.",
  "retry_after": 60
}
```

### 2.6 Implementation Requirements

#### Controller: `LoanTrackingController`

```
app/Http/Controllers/Api/V1/LoanTrackingController.php
```

- Single `show(string $reference)` method
- Validates reference format with regex `^EDF-\d{4}-[A-Z0-9]{6}$` — returns 404 (not 422) on mismatch to avoid revealing validation logic
- Eager-loads the `beneficiary` and `financingPackage` relationships
- Returns `LoanTrackingResource` (never the raw model)
- Catches `ModelNotFoundException` and returns the standard 404 response

#### Resource: `LoanTrackingResource`

```
app/Http/Resources/Api/V1/LoanTrackingResource.php
```

- Explicitly maps only the approved public fields (see Section 2.3)
- Does **not** extend the model's `toArray()` — fields are hand-picked
- If the model gains new fields in the future, they will **not** appear in the tracking response unless explicitly added to this resource

#### Route Registration

In `routes/api.php`, within the public API key middleware group:

```php
Route::middleware(['api.key', 'throttle:tracking,20,1'])
     ->get('/loans/track/{reference}', [LoanTrackingController::class, 'show']);
```

#### Rate Limiter

In `app/Providers/AppServiceProvider.php` (or `RouteServiceProvider`):

```php
RateLimiter::for('tracking', function (Request $request) {
    return Limit::perMinute(20)->by($request->ip());
});
```

#### Data Security & Filtering

The following fields are **explicitly excluded** from the tracking response and are only available in the authenticated Laravel portal:

| Excluded Field | Reason |
|----------------|--------|------|
| `id` (UUID) | Internal identifier — not for public exposure |
| `beneficiary_id` | Internal relationship key |
| `principal` | Financial amount — sensitive |
| `interest_rate` | Financial terms — sensitive |
| `total_amount` | Financial amount — sensitive |
| `amount_paid` | Financial amount — sensitive |
| `outstanding_balance` | Financial amount — sensitive |
| `initial_deposit` | Financial amount — sensitive |
| `deadline` | Repayment schedule detail — sensitive |
| `collateral_id` | Internal relationship key |
| `activated_at` | Internal timestamp |
| `created_at` (full timestamp) | Only the date portion is exposed (as `application_date`) |
| Beneficiary full name | Only initials are exposed (e.g., "John D.") |
| Beneficiary contact details | Not exposed at all |
| Account holder name/email/phone | Not exposed at all |
| Obligor details | Not exposed at all |
| Collateral details | Not exposed at all |
| Document URLs | Not exposed at all |
| Payment history | Not exposed at all |

---

## 3. WordPress Side — Frontend Specification

### 3.1 Page

| Property | Value |
|----------|-------|
| **URL** | `https://edufin.co.ke/track-application` |
| **Slug** | `track-application` |
| **Template** | `page-track-application.php` |
| **Title** | "Track Application" |
| **Auth required** | No (public page) |

### 3.2 Page Template

**File:** `wp-content/themes/edufin/page-track-application.php`

Follows the existing `edufin-resource-page` design pattern (same as calculator, contact, and legal pages):

- **Hero section:** Breadcrumb (Home / Track Application), page title, subtitle
- **Content section:** A centered tracking card containing:
  - Form with a single text input (tracking number), Turnstile widget, and "Track" button
  - Results container (hidden until a lookup is performed)
  - Error container (hidden until an error occurs)
- **CTA section:** "Need help?" with links to Help Center and Contact Us

### 3.3 JavaScript

**File:** `wp-content/themes/edufin/assets/js/track-application.js`

Responsibilities:
1. Read the API base URL from the localized `edufinTracking` object (same pattern as onboarding wizard)
2. Handle form submission — prevent default, validate format client-side, show loading state
3. Call `GET {apiBaseUrl}/loans/track/{reference}` with `X-API-Key` header
4. Render the status card on success (200)
5. Render the error message on failure (404, 429, network error)
6. Reset state on new submission

**API Configuration (in `class-edufin-assets.php`):**

```php
if (is_page_template('page-track-application.php')) {
    wp_enqueue_script('edufin-track-application', $js_uri . 'track-application.js', [], $ver, true);
    wp_localize_script('edufin-track-application', 'edufinTracking', [
        'apiBaseUrl' => rest_url('edufin/v1'),  // Mock during development
        'apiKey'     => EduFin_Settings::get('edufin_public_api_key', ''),
        'turnstileSiteKey' => EduFin_Settings::get('edufin_turnstile_site_key', ''),
    ]);
}
```

> **Production migration:** Change `apiBaseUrl` from `rest_url('edufin/v1')` to `'https://edufin.co.ke/api/v1'` and disable the mock endpoint — identical to the onboarding wizard migration.

### 3.4 Mock REST Endpoint

**File:** `wp-content/themes/edufin/includes/class-edufin-tracking-api.php`

During the current development phase (Laravel not yet built), a mock WordPress REST endpoint serves the same JSON shape:

| Mock Endpoint | Production Endpoint |
|---------------|---------------------|
| `GET /wp-json/edufin/v1/loans/track/{reference}` | `GET /api/v1/loans/track/{reference}` |

The mock returns sample data for a known reference (e.g., `EDF-2026-DEMO01`) and 404 for all others. This allows the UI to be developed and tested before the Laravel backend exists.

**Class structure** mirrors `EduFin_Onboarding_API`:
- `EduFin_Tracking_API` class
- `NAMESPACE = 'edufin/v1'`
- Registered on `rest_api_init`
- `permission_callback: '__return_true'` (public endpoint)

### 3.5 Footer Link Update

The "Track Application" link in the footer is changed from an external portal link to an internal WordPress page link:

**Before:**
```php
<a href="<?php echo esc_url(edufin_url('dashboard')); ?>" target="_blank" rel="noopener noreferrer">
    ↗ Track Application
</a>
```

**After:**
```php
<a href="<?php echo esc_url(home_url('/track-application/')); ?>" class="<?php echo esc_attr(edufin_footer_current('/track-application/')); ?>">
    Track Application
</a>
```

The external icon (↗) is removed since this is now an internal link. The `track-application` slug is added to the `edufin_footer_current()` slug_map for active-state highlighting.

### 3.6 Pages Seeder

Add to `class-edufin-pages-seeder.php`:

```php
'track-application' => [
    'title'   => 'Track Application',
    'content' => '<!-- The tracking UI is rendered by the page-track-application.php theme template. -->',
    'template' => 'page-track-application.php',
    'is_front' => false,
    'is_posts' => false,
],
```

### 3.7 UI States

| State | Trigger | Display |
|-------|---------|---------|
| **Initial** | Page load | Form with input, Turnstile, "Track" button. Hint: "Format: EDF-YYYY-XXXXXX" |
| **Loading** | Form submitted | Spinner + "Looking up your application..." |
| **Success** | API returns 200 | Status card: colored status badge, package name, beneficiary initials, application date, last updated, "View full details" link to portal |
| **Not found** | API returns 404 | "No application found with that tracking number. Please check and try again." |
| **Rate limited** | API returns 429 | "Too many attempts. Please wait a minute and try again." |
| **Network error** | Request fails | "We couldn't reach the tracking service. Please try again later." |

### 3.8 Status Badge Colors

| Status | Color | Label |
|--------|-------|-------|
| `PENDING` | Blue | Pending Review |
| `ACTIVE` | Green | Active |
| `PAID` | Dark green | Fully Paid |
| `DEFAULTED` | Red | Defaulted |
| `RESTRUCTURED` | Orange | Restructured |

---

## 4. Data Classification — Public vs. Portal

### 4.1 Public Tracking Data (WordPress)

The following data is safe to display to anyone who knows the tracking number. It is non-sensitive, contains no PII beyond partial initials, and reveals nothing about financial amounts or account details.

| Field | Sensitivity | Rationale |
|-------|-------------|-----------|
| `reference` | None | The user already knows this — it is the input |
| `status` | Low | A status enum is not sensitive; it tells the user where their application is in the process |
| `status_label` | Low | Human-readable version of the status |
| `package_name` | Low | The financing package name is public (listed on the /products page) |
| `beneficiary_initials` | Low | First name + last initial only — insufficient for identity theft or contact |
| `application_date` | Low | Date only — no time, no timezone, no internal timestamps |
| `last_updated` | Low | Date only — tells the user when the status last changed |
| `portal_url` | None | A public link to the login page |

### 4.2 Sensitive Data — Laravel Portal Only

The following data is **never** exposed through the public tracking endpoint. It requires JWT authentication via the Laravel client portal at `app.edufin.co.ke`.

| Category | Fields | Access |
|----------|--------|--------|
| **Financial amounts** | `principal`, `interest_rate`, `total_amount`, `amount_paid`, `outstanding_balance`, `initial_deposit` | Authenticated (JWT) — `GET /api/v1/loans/{id}` |
| **Repayment schedule** | `deadline`, payment schedule, payment history, statements | Authenticated (JWT) — `GET /api/v1/loans/{id}/schedule`, `/statement` |
| **Personal information** | Account holder full name, email, phone, address, KYC documents | Authenticated (JWT) — `GET /api/v1/profile` |
| **Beneficiary details** | Full name, date of birth, institution, admission number, fee structure | Authenticated (JWT) — `GET /api/v1/beneficiaries/{id}` |
| **Collateral details** | Type, description, estimated value, verified value, documents, lien status | Authenticated (JWT) — `GET /api/v1/collateral/{id}` |
| **Obligor details** | Obligor names, liability shares, assignment history | Authenticated (JWT) — via loan details |
| **Internal identifiers** | UUIDs, `beneficiary_id`, `collateral_id`, `activated_at` (full timestamp) | Never exposed publicly |
| **Documents** | Signed URLs, upload metadata, document content | Authenticated (JWT) — `GET /api/v1/documents/{id}` |

### 4.3 Decision Framework

When deciding whether a field can be added to the public tracking response in the future, apply this test:

1. **Could this field be used to identify, contact, or locate a person?** If yes → portal only.
2. **Does this field reveal financial amounts, rates, or balances?** If yes → portal only.
3. **Is this field an internal identifier (UUID, relationship key)?** If yes → never expose.
4. **Would a competitor find this field useful for market intelligence?** If yes → consider keeping portal-only.
5. **Is the field already public on the marketing site?** If yes → safe to include (e.g., package name).

---

## 5. Security Considerations

### 5.1 Enumeration Prevention

| Control | Implementation |
|---------|---------------|
| **Reference format** | `EDF-YYYY-XXXXXX` where XXXXXX is 6 random alphanumeric characters (36^6 ≈ 2.2 billion combinations) |
| **Rate limiting** | 20 requests per minute per IP (Laravel `throttle:tracking` limiter) |
| **Turnstile** | Cloudflare Turnstile widget on the WordPress form — blocks automated submissions |
| **Uniform 404** | Not-found, malformed, and deleted records all return the same 404 response |
| **No metadata** | Response includes no internal IDs, timestamps (beyond date), or error details that could aid enumeration |

### 5.2 API Key Security

The `X-API-Key` used by this endpoint is the same public key used by the onboarding wizard's `/options/*` endpoints. It is sent from the browser and is therefore visible in network traffic. This is acceptable because:

- The key only grants access to public endpoints (packages, options, tracking, inquiries, newsletter)
- All authenticated endpoints require a JWT token, which is never exposed to the public
- The key is not a secret in the cryptographic sense — it is an identifier for rate-limit grouping and abuse tracking

### 5.3 WordPress Security Boundary

WordPress never receives sensitive data through this feature. The tracking response contains only the whitelisted public fields. Even if WordPress is fully compromised:

- The attacker cannot access financial amounts, personal information, or documents
- The attacker cannot enumerate references faster than the Laravel rate limiter allows
- The attacker cannot modify the data — the tracking endpoint is read-only

### 5.4 Data Protection Compliance

This feature is compliant with the Kenya Data Protection Act, 2019, because:

- The only personal data exposed is beneficiary initials (first name + last initial) — this is not sufficient to identify an individual
- The lookup requires knowledge of the reference number, which is provided to the applicant at application time
- No new data is collected from the user (only the reference number they already have)
- The feature does not set any cookies beyond the standard site cookies
- The response is not logged with the reference number in WordPress (the call is client-side)

---

## 6. Implementation Phases

### Phase 1: WordPress Mock (Current — Laravel not yet built)

| Step | Deliverable |
|------|-------------|
| W1 | Create `page-track-application.php` theme template |
| W2 | Create `assets/js/track-application.js` |
| W3 | Create mock `EduFin_Tracking_API` class (WordPress REST endpoint) |
| W4 | Enqueue script + localize API config in `class-edufin-assets.php` |
| W5 | Add page to pages seeder |
| W6 | Update footer "Track Application" link to internal page |
| W7 | Add `track-application` to `edufin_footer_current()` slug_map |
| W8 | Add Turnstile widget to the form |
| W9 | Seed page via `wp edufin seed pages` |

### Phase 2: Laravel API (When Laravel is built)

| Step | Deliverable |
|------|-------------|
| L1 | Create `LoanTrackingController` with `show()` method |
| L2 | Create `LoanTrackingResource` (field-whitelisted) |
| L3 | Register route in `routes/api.php` with API key + tracking rate limiter |
| L4 | Configure `throttle:tracking` rate limiter (20/min/IP) |
| L5 | Test with real loan facilities in the database |

### Phase 3: Production Migration

| Step | Deliverable |
|------|-------------|
| M1 | Update `apiBaseUrl` in `class-edufin-assets.php` from `rest_url('edufin/v1')` to `https://edufin.co.ke/api/v1` |
| M2 | Disable or remove the mock `EduFin_Tracking_API` class |
| M3 | Verify the Laravel endpoint returns the exact same JSON shape as the mock |
| M4 | End-to-end test on production |

---

## 7. File Inventory

### WordPress (to be created)

| File | Purpose |
|------|---------|
| `wp-content/themes/edufin/page-track-application.php` | Page template |
| `wp-content/themes/edufin/assets/js/track-application.js` | Frontend logic |
| `wp-content/themes/edufin/includes/class-edufin-tracking-api.php` | Mock REST endpoint (Phase 1 only) |

### WordPress (to be modified)

| File | Change |
|------|--------|
| `wp-content/themes/edufin/footer.php` | Change "Track Application" link to internal page; add slug to `edufin_footer_current()` |
| `wp-content/themes/edufin/includes/class-edufin-assets.php` | Enqueue tracking script + localize API config |
| `wp-content/plugins/edufin-core/seeders/class-edufin-pages-seeder.php` | Add `track-application` page entry |

### Laravel (to be created — Phase 2)

| File | Purpose |
|------|---------|
| `app/Http/Controllers/Api/V1/LoanTrackingController.php` | API controller |
| `app/Http/Resources/Api/V1/LoanTrackingResource.php` | Field-whitelisted resource |
| `routes/api.php` (modified) | Route registration |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | August 19, 2026 | EduFin Team | Initial specification |
