---
description: "Use when designing REST endpoints, defining Pydantic request/response schemas, configuring middleware, or planning integration contracts with external services."
---

# PatriotForge API Conventions

**Stack:** FastAPI · Pydantic v2 · Redis sessions · CSRF protection · Rate limiting

## RESTful Endpoint Design
| Action | Verb | Status | Example |
|--------|------|--------|---------|
| Create | POST | 201 | `POST /api/quotes` |
| List | GET | 200 | `GET /api/quotes?page=1&per_page=25` |
| Detail | GET | 200 | `GET /api/quotes/{id}` |
| Update | PATCH | 200 | `PATCH /api/quotes/{id}` |
| Delete | DELETE | 204 | `DELETE /api/quotes/{id}` (soft delete) |

Always declare `response_model` and `status_code` on every endpoint.

## Pydantic Schema Patterns
```python
# Request — reject unknown fields
class CreateQuoteRequest(BaseModel):
    model_config = ConfigDict(extra='forbid')
    customer_id: UUID
    notes: str = Field(max_length=2000, default="")

# Response — ORM-compatible
class QuoteResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: UUID
    customer_id: UUID
    status: str
    created_at: datetime
```

## Paginated List Response
```python
class PaginatedResponse(BaseModel, Generic[T]):
    total: int
    page: int
    per_page: int
    items: list[T]
```
Default: `page=1`, `per_page=25`, max `per_page=100`.

## Authentication & Security Middleware
- **Session cookie:** HTTP-only, Secure, SameSite=Lax — set on login
- **CSRF:** `X-CSRF-Token` header required on POST/PATCH/DELETE
- **Rate limiting:** 5 attempts/min per IP on auth endpoints; 10/min per account
- **CORS:** Single allowed origin `https://forge.patriotpf.com`, credentials enabled
- **Security headers:** `Cache-Control: no-store` on auth responses

## Error Response Format
```json
{
  "detail": "Human-readable message",
  "code": "DUPLICATE_EMAIL",
  "correlation_id": "uuid"
}
```
- Never expose stack traces, internal paths, or SQL errors
- Log full details server-side with correlation ID

## Integration Contracts

| System | Protocol | Auth | Key Pattern |
|--------|----------|------|-------------|
| Floor Tracker | HTTPS REST | API key | Push WOs, poll status |
| ShipStation | REST v2 | API key | Rate shop, create shipments |
| Stripe | Checkout Sessions | Secret key | Webhooks w/ signature verification |
| QuickBooks | REST + OAuth 2.0 | OAuth tokens | Batch export queue |
| OnPrintShop | Zapier webhook | Shared secret | Inbound SO creation |
| Email (M365) | Graph API | OAuth | Claude-parsed → pending_review quote |
| Artworker.io | Webhooks | Signature | Proof approval status |

All webhooks: verify signature → deduplicate by event ID → return 200 immediately → process async.

📖 **Full details:** `backend/app/routers/auth.py`, `docs/plans/2026-01-24-integrations.md`
