# API Documentation

## EduFin REST API

**Base URL:** `https://edufin.co.ke/api/v1`

## Authentication

| Endpoint Type | Method | Header |
|---------------|--------|--------|
| Public | API Key | `X-API-Key: {key}` |
| Authenticated | JWT | `Authorization: Bearer {token}` |
| Webhooks | Signature | `X-Webhook-Signature: {sig}` |

## Endpoints

See detailed documentation:
- [Authentication](./authentication.md)
- [Clients](./clients.md)
- [Loans](./loans.md)
- [Webhooks](./webhooks.md)

## Rate Limits

| Endpoint Type | Limit |
|---------------|-------|
| Public | 100 req/min |
| Authenticated | 60 req/min per user |
| Webhooks | Unlimited |
