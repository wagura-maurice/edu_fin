# Loan API

Endpoints for loan operations.

## Public Tracking Endpoint

### GET `/loans/track/{reference}`

Public lookup of a loan application's status by tracking number. **No authentication required** (API key only). Returns a whitelisted subset of non-sensitive fields only.

> **See:** [Public Loan Tracking Feature](../features/public-loan-tracking.md) for full specification, data classification, and security considerations.

**Headers:**

| Header | Required | Description |
|--------|----------|-------------|
| `X-API-Key` | Yes | Public API key |
| `Accept` | No | `application/json` (default) |

**URL Parameter:**

| Parameter | Format | Description |
|-----------|--------|-------------|
| `reference` | `EDF-YYYY-XXXXXX` | The `facility_reference` to look up |

**Rate Limit:** 20 requests per minute per IP address.

**Response (200 OK):**

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

**Response (404 Not Found):**

```json
{
  "success": false,
  "message": "No application found with that tracking number. Please check and try again."
}
```

> The 404 response is identical for not-found, malformed, and deleted records — the API never confirms whether a reference exists.

**Status Values:**

| Status | Label | Description |
|--------|-------|-------------|
| `PENDING` | Pending Review | Application submitted, awaiting review |
| `ACTIVE` | Active | Loan approved and disbursed |
| `PAID` | Fully Paid | Loan fully repaid |
| `DEFAULTED` | Defaulted | Payments overdue beyond contract period |
| `RESTRUCTURED` | Restructured | Loan terms modified |

---

## Authenticated Loan Endpoints

The following endpoints require JWT authentication (`Authorization: Bearer {token}`) and return the full loan data. These are consumed by the Laravel client portal and future mobile apps — **not** by the WordPress public tracking page.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/loans` | List the authenticated user's loans |
| POST | `/loans` | Apply for a new loan |
| GET | `/loans/{id}` | Full loan details (all fields) |
| GET | `/loans/{id}/schedule` | Payment schedule |
| GET | `/loans/{id}/statement` | Generate statement |

> **Data classification:** The public tracking endpoint (`GET /loans/track/{reference}`) returns only non-sensitive status fields. The authenticated endpoints above return the full Loan Facility model including financial amounts, repayment schedule, and related entity details. See [Public Loan Tracking — Data Classification](../features/public-loan-tracking.md#4-data-classification--public-vs-portal) for the complete field-by-field breakdown.

*Additional authenticated loan endpoint details to be completed.*
