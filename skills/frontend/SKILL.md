---
description: "Use when writing React/TypeScript frontend code — components, pages, routing, styling, layout patterns, or UI primitives."
---

# PatriotForge Frontend Patterns

**Stack:** React 19 · TypeScript (strict) · Vite · Tailwind CSS v4 · React Router v7

## Directory Structure
```
prototype/src/
├── App.tsx              # Router config — all 20+ routes
├── components/
│   ├── layout/          # Layout, Sidebar, TopBar
│   ├── ui/              # Shared primitives (Card, Badge, Button, StatCard)
│   └── [module]/        # Module-specific components
├── pages/               # Route-level page components
├── types/               # TypeScript interfaces (mirror DB schema)
├── data/                # Mock data (TS objects matching types)
└── hooks/               # Custom React hooks
```

## Component Conventions
- Functional components only — no class components
- TypeScript interfaces for all props (`interface Props { ... }`)
- Named exports: `export function QuoteList() { ... }`
- Barrel exports: `index.ts` per module folder

## Layout Pattern
```
┌─────────────────────────────────┐
│           TopBar                │
├──────┬──────────────────────────┤
│      │                          │
│ Side │      <Outlet />          │
│ bar  │    (page content)        │
│      │                          │
└──────┴──────────────────────────┘
```
- `Layout.tsx` wraps all routes with Sidebar + TopBar + React Router `<Outlet />`
- Sidebar: navigation links grouped by module
- TopBar: search, notifications, user menu

## Routing (React Router v7)
```tsx
<Route path="/" element={<Layout />}>
  <Route index element={<Dashboard />} />
  <Route path="quotes" element={<QuoteList />} />
  <Route path="quotes/:id" element={<QuoteDetail />} />
  <Route path="quotes/:id/edit" element={<QuoteBuilder />} />
</Route>
```
- Resource pattern: `list → detail → edit`
- All routes nested under `<Layout />` wrapper

## Styling (Tailwind v4)
- Utility classes only — no custom CSS files
- Color palette: defined in Tailwind config (brand blues, status colors)
- Responsive breakpoints: Desktop ≥1200px, Tablet 768–1199px, Mobile <768px
- Dark mode: not yet implemented, plan for `dark:` variants

## UI Primitives
- `Card` — content container with header/body/footer
- `Badge` — status indicators (color-coded by state)
- `Button` — primary, secondary, danger, ghost variants
- `StatCard` — KPI display (value, label, trend icon)
- `Modal` — confirmation dialogs, quick-add forms, selection pickers

## Page Layout Patterns
- **List View:** tabs + search + filters + bulk actions + sortable table
- **Detail View:** vertical single-column with header, sections, activity feed
- **Builder Form:** auto-save every 30s, dynamic fields per product type, live pricing
- **Dashboard:** customizable widget grid, role-based visibility
- **CRM Page:** tabbed contact view (orders, quotes, invoices, notes)

## Mock Data
- TypeScript interfaces mirror the database schema exactly
- Mock data files provide realistic test data for UI development

📖 **Reference:** `prototype/src/App.tsx`, `prototype/src/components/layout/`, `docs/plans/2026-01-24-page-layouts.md`
