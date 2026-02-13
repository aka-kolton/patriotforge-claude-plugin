---
description: "Use when reviewing code changes, pull requests, or auditing existing code for compliance with PatriotForge conventions, security rules, and quality standards."
---

# PatriotForge Code Review Checklist

Run through every category. Flag security violations as **🔴 CRITICAL**. Flag convention mismatches as **🟡 WARNING**.

## 1. Security Compliance
- [ ] No plaintext passwords, no secrets in code/logs
- [ ] Parameterized queries only — no SQL string building
- [ ] Pydantic `extra='forbid'` on all request schemas
- [ ] Input validation: max lengths, magic bytes on uploads
- [ ] CSRF token required on state-changing endpoints
- [ ] Webhook signatures verified before processing
- [ ] No `localStorage` for secrets — HTTP-only cookies only
- [ ] Stripe: Checkout Sessions only, webhook sig verified
→ Full checklist: invoke `/patriot-forge:security` or read `docs/SECURITY_RULES.md`

## 2. Backend Conventions
- [ ] All functions async — no sync DB or Redis calls
- [ ] Service layer has no HTTP objects (no `Request`, `Response`, `HTTPException`)
- [ ] Domain exceptions in services, HTTP mapping in routers
- [ ] `response_model` and `status_code` on every endpoint
- [ ] Dependency injection via `Depends()` — no global state

## 3. Frontend Conventions
- [ ] TypeScript strict mode — no `any` without justification
- [ ] Functional components with typed props interfaces
- [ ] Tailwind utilities only — no custom CSS
- [ ] Named exports, barrel `index.ts` per module
- [ ] Permission checks mirror backend (defense in depth)

## 4. Database Conventions
- [ ] `forge_` table prefix on all new tables
- [ ] UUID primary keys — no auto-increment
- [ ] `NUMERIC(12,2)` for money — never `FLOAT`
- [ ] Soft delete (`deleted_at`) — never hard delete
- [ ] `created_at`, `updated_at` timestamps on all tables
- [ ] Alembic migration included for schema changes

## 5. Type Safety
- [ ] Strict mypy passes (backend) — no `# type: ignore` without comment
- [ ] TypeScript strict (frontend) — no implicit `any`
- [ ] Pydantic models match DB schema (field names, types)

## 6. Test Coverage
- [ ] New features have tests (service + API level)
- [ ] Tests cover: success, 422, 401, 429, 409 cases
- [ ] No mocking of internal layers — test through real service
- [ ] Cookie attributes and security headers verified

## 7. CI/CD Compatibility
- [ ] `ruff check` and `ruff format` pass
- [ ] `mypy --strict` passes
- [ ] `bandit` — no new findings
- [ ] No hardcoded secrets (would fail gitleaks)

## 8. Git & Style
- [ ] Commit message: `fix:`, `feat:`, `chore:`, or `refactor:` prefix
- [ ] Changes scoped to one concern per commit
- [ ] No `sed`/`awk` in scripts — use Python for file editing

📖 **Cross-references:** All other PatriotForge skills; `docs/SECURITY_RULES.md`
