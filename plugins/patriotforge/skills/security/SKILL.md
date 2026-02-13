---
description: "Use when writing or reviewing security-sensitive code — authentication, payments, webhooks, input validation, secrets management, CORS/CSRF, file uploads, or audit logging."
---

# PatriotForge Security Checklist

Enforce these rules on every code change. Flag violations as **CRITICAL**.

## 1. Authentication
- Argon2id password hashing (12–128 char passwords)
- Redis-backed sessions — NEVER JWT for auth
- Session token stored as `session:{HMAC-SHA256(token, secret)}` in Redis
- HTTP-only, Secure, SameSite=Lax cookies only
- 30-min idle timeout, 14-day absolute lifetime
- Rotate session on login, privilege change, MFA success

## 2. Authorization
- All permission checks in backend middleware — NEVER trust frontend
- Scoped queries: filter by `company_id` + `deleted_at IS NULL`
- Log every denied access attempt

## 3. CSRF / CORS
- `X-CSRF-Token` header required on all state-changing requests
- CORS origin: `https://forge.patriotpf.com` only — no wildcards
- Validate `Origin` header server-side

## 4. Database
- Parameterized queries ONLY — no f-strings or concatenation in SQL
- Runtime role: `forge_app` (least privilege); migrations: `forge_migrate`
- Soft delete (`deleted_at`) — NEVER hard delete
- Money: `NUMERIC(12,2)` — NEVER float

## 5. Secrets
- Environment variables only — no `.env` in repo, no secrets in logs
- OAuth tokens encrypted at rest (AES-256-GCM)

## 6. Stripe
- Checkout Sessions only — NEVER handle raw card data
- Verify webhook signatures (`stripe-signature` header)
- Idempotent payment handlers; MFA required for refunds

## 7. Webhooks
- Verify signatures before processing
- Rate limit inbound webhooks
- Idempotent handlers (deduplicate by event ID)
- Return 200 immediately, process async

## 8. Input Validation
- Pydantic `extra='forbid'` on all request schemas
- Enforce max lengths on all string fields
- Validate file magic bytes, not just extensions

## 9. File Uploads
- Magic byte validation + UUID filenames
- Store outside web root, serve via authenticated endpoint
- 50 MB max size

## 10. Error Handling
- No stack traces in responses — generic messages to client
- Structured JSON errors with correlation IDs
- Log full details server-side only

## 11. Frontend
- No secrets in localStorage — HTTP-only cookies only
- Send `X-CSRF-Token` header on every mutation
- Never embed user input as raw HTML

## 12. Deployment
- Railway env vars for all secrets — private networking between services
- Trivy container scan on every build

## 13. CI/CD Security Gates
- gitleaks (secrets), bandit (Python), pip-audit, npm audit, Trivy
- Fail pipeline on HIGH or CRITICAL findings

## 14. Audit Trail
- Log all Create/Update/Delete on financial records
- NEVER delete audit log entries — append only

📖 **Full details:** `docs/SECURITY_RULES.md`, `docs/plans/security-plan.md`
