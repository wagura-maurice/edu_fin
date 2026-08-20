# Authentication API

Endpoints for user authentication and registration.

**Base URL:** `https://edufin.co.ke/api/v1`

## Registration (API-Only — No Front-End Page)

> **Important:** The Laravel application does **not** host a front-end `/register` page. Registration is handled exclusively via this API endpoint. The WordPress onboarding wizard (`edufin.co.ke/get-started`) is the primary consumer; future mobile applications will consume the same endpoint.

### POST `/auth/register`

Create a new client account.

**Headers:**

| Header | Required | Description |
|--------|----------|-------------|
| `X-API-Key` | Yes | Public API key |
| `Content-Type` | Yes | `application/json` |

**Request Body:**

```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "email": "jane.doe@example.com",
  "phone": "+254700000000",
  "gender_id": 1,
  "location_id": 14,
  "employment_type_id": 2,
  "employer_name": "Acme Ltd",
  "income_range_id": 3,
  "beneficiary_name": "John Doe",
  "relationship_type_id": 1,
  "school_name": "Nairobi School",
  "education_level_id": 3,
  "password": "securepassword",
  "password_confirmation": "securepassword",
  "terms_accepted": true
}
```

**Response (201 Created):**

```json
{
  "success": true,
  "data": {
    "account_id": 12345,
    "email": "jane.doe@example.com",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
  }
}
```

**Response (422 Unprocessable Entity):**

```json
{
  "success": false,
  "errors": {
    "email": ["The email has already been taken."],
    "phone": ["The phone format is invalid."]
  }
}
```

## Dynamic Options (For Onboarding Wizard)

These endpoints provide the dropdown options consumed by the WordPress onboarding wizard. They are public (API key required).

| Method | Endpoint | Returns |
|--------|----------|---------|
| GET | `/options/locations` | List of counties/cities |
| GET | `/options/genders` | List of gender options |
| GET | `/options/employment-types` | List of employment types |
| GET | `/options/income-ranges` | List of income ranges |
| GET | `/options/education-levels` | List of education levels |
| GET | `/options/relationship-types` | List of beneficiary relationship types |

**Example Response (200 OK):**

```json
{
  "success": true,
  "data": [
    { "id": 1, "label": "Nairobi", "value": "nairobi" },
    { "id": 2, "label": "Mombasa", "value": "mombasa" },
    { "id": 3, "label": "Kisumu", "value": "kisumu" }
  ]
}
```

## Login

### POST `/auth/login`

Authenticate a user and receive a JWT token.

**Request Body:**

```json
{
  "email": "jane.doe@example.com",
  "password": "securepassword"
}
```

**Response (200 OK):**

```json
{
  "success": true,
  "data": {
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "user": {
      "id": 12345,
      "name": "Jane Doe",
      "email": "jane.doe@example.com",
      "role": "client"
    }
  }
}
```

## Password Reset

### POST `/auth/forgot-password`

Request a password reset link.

### POST `/auth/reset-password`

Reset password using the token from the email link.

## Token Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/refresh` | Refresh an expired JWT token |
| POST | `/auth/logout` | Invalidate the current token |
| POST | `/auth/verify-otp` | Verify phone/email OTP |

## Laravel Web UI (Non-API)

The Laravel web UI is intentionally limited to these pages only (no `/register` page):

| Route | Purpose |
|-------|---------|
| `app.edufin.co.ke/login` | Login form (role-based redirect) |
| `app.edufin.co.ke/forgot-password` | Password reset request |
| `app.edufin.co.ke/password/reset` | Password reset form |
| `app.edufin.co.ke/logout` | End session |
