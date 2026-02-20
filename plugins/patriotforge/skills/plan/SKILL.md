---
description: "Use to investigate and plan a feature, bug fix, or architectural change. Dispatches fast scouts (haiku) for recon, then planners (sonnet) for architecture. Searches codebase, server, Railway, and web."
argument-hint: Feature idea, bug, or investigation topic
---

# PatriotForge Plan & Investigate

Structured investigation and planning pipeline. Takes a vague idea and produces a phased, actionable implementation plan.

`$ARGUMENTS` — the feature, bug, or topic to investigate

---

## Phase 1: Discovery

**Goal:** Understand what the user wants

1. If `$ARGUMENTS` is provided, summarize your understanding
2. If vague or missing, ask with `AskUserQuestion`:
   - What problem are you solving?
   - What should the end result look like?
   - Any constraints (timeline, scope, dependencies)?
3. Confirm understanding before proceeding

---

## Phase 2: Reconnaissance (Scouts — haiku)

**Goal:** Gather facts fast using cheap, parallel scouts

Launch **3-5 scout agents in parallel** (`subagent_type: "patriotforge:scout"`, model will default to haiku). Each scout investigates a different angle:

### Scout Assignments (pick the most relevant):

**Codebase Scout** — "Search the PatriotForge codebase for anything related to {topic}. Find existing models, schemas, routers, services, and components. Check `D:/PatriotForge/backend/app/` and `D:/PatriotForge/frontend/src/`. Report file paths, key patterns, and relevant code."

**Similar Feature Scout** — "Find the most similar existing feature to {topic} in PatriotForge. Trace its full implementation: model → schema → service → router → API client → React component. Report the pattern so we can replicate it."

**Database Scout** — "Check the current database schema related to {topic}. Look at models in `D:/PatriotForge/backend/app/models/`, check existing migrations in `alembic/versions/`, and report table structures, relationships, and any relevant constraints."

**Server Scout** — "SSH to `patriotdev@100.111.104.5` and check the running state related to {topic}. Check container status (`docker ps`), recent logs, database state (`psql` queries against `printshop`), and any relevant configuration."

**API/Integration Scout** — "Research the external API or service needed for {topic}. Check docs for {Stripe/ShipStation/QuickBooks/etc.}. Report: endpoints we need, auth method, data format, rate limits, webhooks."

**Frontend Scout** — "Search `D:/PatriotForge/frontend/src/` for existing UI patterns related to {topic}. Check features/, components/, api/ clients, and routing. Report component patterns, state management approach, and relevant hooks."

**Test Scout** — "Check existing tests in `D:/PatriotForge/backend/tests/` related to {topic}. Report test patterns, fixtures used, and what coverage exists. Also check if there are any failing tests."

**Plan/Status Scout** — "Read existing plans and status trackers at `C:\Users\Kolton\Documents\Logbook\Projects\PatriotForge\` related to {topic}. Check Plans/, Docs/Status Trackers/, and TODO.md for any prior work, decisions, or open items."

### Model Selection Guide

| Task | Model | Why |
|------|-------|-----|
| File search, grep, list files | haiku (scout) | Fast, cheap, just finding things |
| Read + summarize code patterns | haiku (scout) | Pattern recognition doesn't need deep reasoning |
| SSH commands, docker status | haiku (scout) | Simple command execution |
| Web doc fetching | haiku (scout) | Just retrieving and summarizing |
| Deep architectural analysis | sonnet (planner) | Needs reasoning about trade-offs |
| Plan production | sonnet (planner) | Needs to synthesize and make decisions |

---

## Phase 3: Read Key Files

**Goal:** Build deep context from scout findings

1. Collect all scout reports
2. Read the **5-10 most important files** identified by scouts
3. Summarize findings to the user:
   - What exists already
   - What patterns to follow
   - What's missing
   - Any blockers or surprises

---

## Phase 4: Clarifying Questions

**Goal:** Resolve ambiguities before planning

**DO NOT SKIP THIS PHASE.**

1. Based on findings, identify:
   - Scope decisions (what's in/out)
   - Architecture choices (if multiple valid approaches)
   - Integration unknowns
   - Edge cases that need user input
2. Present questions in a clear list using `AskUserQuestion`
3. Wait for answers before proceeding

If the user says "your call" — make the decision, state it clearly, and proceed.

---

## Phase 5: Architecture & Planning (Planner — sonnet)

**Goal:** Produce a decisive implementation plan

Launch **1-2 planner agents** (`subagent_type: "patriotforge:planner"`) with:
- All scout findings summarized
- User's answers to clarifying questions
- The specific feature/topic to plan
- Key files to read for pattern reference

If the feature is large, launch 2 planners with different focuses:
- **Backend planner:** models, schemas, services, routers, migrations, tests
- **Frontend planner:** components, pages, API clients, routing, state management

---

## Phase 6: Plan Output

**Goal:** Deliver the final plan

1. Synthesize planner output into a single cohesive plan
2. Present to the user with:
   - Overview (1-2 sentences)
   - Phase breakdown (numbered, with agents and files)
   - Key architecture decisions
   - Open questions (if any remain)
3. Save the plan to: `C:\Users\Kolton\Documents\Logbook\Projects\PatriotForge\Plans\[ready] {Feature Name}.md`
4. Ask: "Plan saved. Ready to implement, or want to adjust anything?"

---

## Ground Rules

- **Scouts are cheap** — use 3-5 in parallel, don't be stingy
- **Planners are deliberate** — use 1-2, give them rich context
- **Never skip clarifying questions** — assumptions kill plans
- **Be decisive** — the plan should have ONE approach, not a menu of options
- **Include everything** — migrations, tests, API endpoints, frontend components, error handling
- **Use forward slashes** in Bash paths (`D:/PatriotForge/...`)
- **Plans go to Obsidian** — `C:\Users\Kolton\Documents\Logbook\Projects\PatriotForge\Plans\`
- **Don't implement** — this skill produces plans only
