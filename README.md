# PatriotForge Claude Code Plugin

Full-stack development orchestration for [PatriotForge ERP](https://github.com/aka-kolton/patriotforge) — 12 specialized agents and 8 domain skills for Claude Code.

## What's Included

### 12 Specialized Agents

| Agent | Model | Role |
|-------|-------|------|
| **patriotdevbot** | opus | Orchestrator — reads plans, breaks into phases, dispatches agents, manages PRs |
| **backend-dev** | sonnet | FastAPI routers, SQLAlchemy models, Pydantic schemas, service functions |
| **database-dev** | sonnet | SQLAlchemy models, Alembic migrations, schema design |
| **api-dev** | sonnet | Pydantic schemas, endpoint contracts, middleware configuration |
| **tdd-agent** | sonnet | pytest-asyncio tests, fixtures, coverage verification |
| **frontend-dev** | sonnet | React 19 components, TypeScript, TanStack Query, routing |
| **ui-agent** | sonnet | TailwindCSS styling, component design, responsive layouts |
| **ux-agent** | sonnet | User flows, accessibility, interaction patterns |
| **security-reviewer** | opus | Security audits, auth review, vulnerability detection (read-only) |
| **code-reviewer** | opus | Convention compliance, quality review, silent failure detection (read-only) |
| **railway-agent** | sonnet | Docker, Railway deployment, CI/CD, database migrations |
| **github-agent** | sonnet | Git workflow, PR creation, branch management, CI monitoring |

### 8 Domain Skills

| Skill | Loaded When |
|-------|-------------|
| **backend** | Writing FastAPI routers, services, config |
| **frontend** | Writing React components, pages, styling |
| **database** | Designing tables, writing migrations |
| **api** | Designing endpoints, Pydantic schemas |
| **security** | Reviewing auth, payments, webhooks |
| **tdd** | Writing tests, fixtures, coverage |
| **code-review** | Reviewing PRs, auditing code quality |
| **deploy** | Docker, CI/CD, Railway deployment |

## Installation

### From GitHub (recommended)

```bash
# Step 1: Add the marketplace
/plugin marketplace add aka-kolton/patriotforge-claude-plugin

# Step 2: Install the plugin
/plugin install patriotforge@patriotforge-marketplace
```

### From local directory (development)

```bash
claude --plugin-dir ./plugins/patriotforge
```

## Usage

After installation, agents and skills are namespaced under `patriotforge:`:

```
# Invoke a skill directly
/patriotforge:backend

# Use an agent
Use the patriotforge:backend-dev agent to implement the new router

# Run the orchestrator
Use the patriotforge:patriotdevbot agent to implement the plan at /path/to/plan.md
```

## Tech Stack

This plugin is built for the PatriotForge ERP tech stack:

| Layer | Technology |
|-------|-----------|
| Backend | FastAPI (Python 3.12), async SQLAlchemy 2.0, Alembic, Pydantic v2 |
| Frontend | React 19, TypeScript (strict), Vite, TailwindCSS v4, React Router v7 |
| Database | PostgreSQL 15 |
| Cache | Redis 7 |
| Auth | Redis sessions, Argon2 passwords, TOTP MFA |
| Deployment | Railway, Docker, GitHub Actions CI |

## Project Structure

```
patriotforge-claude-plugin/
├── .claude-plugin/
│   └── marketplace.json       # Marketplace catalog
├── plugins/
│   └── patriotforge/
│       ├── .claude-plugin/
│       │   └── plugin.json    # Plugin manifest
│       ├── agents/            # 12 specialized agents
│       │   ├── patriotdevbot.md
│       │   ├── backend-dev.md
│       │   ├── database-dev.md
│       │   ├── api-dev.md
│       │   ├── tdd-agent.md
│       │   ├── frontend-dev.md
│       │   ├── ui-agent.md
│       │   ├── ux-agent.md
│       │   ├── security-reviewer.md
│       │   ├── code-reviewer.md
│       │   ├── railway-agent.md
│       │   └── github-agent.md
│       └── skills/            # 8 domain skills
│           ├── backend/SKILL.md
│           ├── frontend/SKILL.md
│           ├── database/SKILL.md
│           ├── api/SKILL.md
│           ├── security/SKILL.md
│           ├── tdd/SKILL.md
│           ├── code-review/SKILL.md
│           └── deploy/SKILL.md
├── README.md
├── CHANGELOG.md
└── LICENSE
```

## Customizing for Your Project

The agents reference PatriotForge-specific paths and conventions. To adapt for your own project:

1. Fork this repo
2. Update file paths in agent prompts (search for `D:\PatriotForge`)
3. Update conventions (table prefixes, tech stack, etc.)
4. Update skill content to match your project patterns

## License

MIT
