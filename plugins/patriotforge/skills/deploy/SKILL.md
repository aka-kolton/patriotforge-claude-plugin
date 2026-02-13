---
description: "Use when deploying, configuring CI/CD pipelines, writing Dockerfiles, running database migrations, bumping versions, or managing the release workflow."
---

# PatriotForge Deployment & CI/CD

**Platform:** Railway (auto-deploy from `main`) · Docker multi-stage builds · GitHub Actions CI

## Railway Deployment
- Auto-deploy triggers on push to `main`
- Custom domain: `forge.patriotpf.com`
- All secrets via Railway environment variables — never in code
- Private networking between backend, PostgreSQL, Redis
- PostgreSQL 15 (shared `printshop` database) + Redis 7

## Docker (Multi-Stage Build)
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ARG VITE_BASE_PATH=/
RUN npm run build

# Stage 2: Serve
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
```
- Backend: Python 3.12 slim base image
- Frontend: Node 20 Alpine → nginx Alpine
- Keep images minimal — no dev dependencies in production

## CI Pipeline (`.github/workflows/ci.yml`)

```
gitleaks (secrets scan)
    ├── Backend: ruff → mypy → bandit → pip-audit → pytest
    ├── Frontend: eslint → tsc → npm audit
    └── Trivy (container + dependency scan)
```

**Gate rules:**
- gitleaks must pass before any other job runs
- bandit: fail on any finding
- pip-audit / npm audit: fail on HIGH+
- Trivy: fail on HIGH or CRITICAL severity
- pytest: continue-on-error during TDD phase (tests written before implementation)

## Database Migrations
- Generate: `alembic revision --autogenerate -m "description"`
- Apply on deploy: `alembic upgrade head`
- Use `forge_migrate` role for DDL operations
- Always import all models in `alembic/env.py`

## Version Bumping
- Semantic versioning: `MAJOR.MINOR.PATCH`
- **Increment PATCH** on every container rebuild
- **Increment MINOR** for new features
- **Increment MAJOR** for breaking changes
- Version lives in: `frontend/package.json` → `"version": "x.y.z"`
- Update `CHANGELOG.md` before every rebuild

## Git Commit Style
| Prefix | Use |
|--------|-----|
| `fix:` | Bug fixes |
| `feat:` | New features |
| `chore:` | Maintenance, rebuilds |
| `refactor:` | Code improvements |

## Important Rules
- **Never use `sed` or `awk`** for file editing — use Python scripts
- Always check logs after rebuild: `docker-compose logs -f [container]`
- Run migrations after backend deploys
- Trivy scan before pushing images

📖 **Reference:** `prototype/Dockerfile.prod`, `.github/workflows/ci.yml`, `docs/SECURITY_RULES.md` (CI/CD section)
