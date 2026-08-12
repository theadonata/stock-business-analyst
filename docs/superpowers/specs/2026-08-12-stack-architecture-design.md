# Stock/HPP App — Stack & Architecture Design

Date: 2026-08-12

## Purpose

Replace the existing Excel-based tracking (`Idea/Catatan_HPP_Keuangan_Bisnis.xlsx`)
with a web app for a small business (bags/accessories) to track sales, COGS
(HPP), operational expenses, inventory, and profit/loss. Used internally by
the owner and a small staff team, including mobile-first entry from the
warehouse floor.

## Scope assumptions

- Internal use only (owner + staff), not multi-tenant SaaS.
- Mobile-first UI needed for stock entry; desktop is fine for reporting.
- Python backend ecosystem preferred.
- Self-hosted deployment on a Kubernetes cluster running on a VPS, with
  each service built as an independent Docker image.
- Always-online; no offline support needed for now.

## Architecture

Two independently deployable services communicating over a versioned REST
API, matching the existing repo split:

- **`stock-backend`**: FastAPI + PostgreSQL. Exposes `/api/v1/...` JSON REST
  endpoints. OpenAPI docs auto-generated at `/docs`, serving as the API
  contract for `stock-frontend` and `stock-qa`.
- **`stock-frontend`**: React + Vite + TypeScript SPA, Tailwind CSS for
  styling, TanStack Query for API data fetching/caching. Built to static
  files, served behind the reverse proxy. Single responsive layout serving
  both mobile and desktop (see UI/UX design below).
- **`stock-infrastructure`**: Kubernetes manifests (Deployments, Services,
  Ingress, ConfigMaps/Secrets) targeting a self-hosted k8s cluster on the
  VPS. `stock-backend` and `stock-frontend` each build and publish their
  own standalone Docker image independently (no shared `docker-compose.yml`
  coupling them) — each repo owns a `Dockerfile` and can be built/deployed
  on its own, consistent with the "no shared code or path dependency"
  convention across the `stock-*` repos. `stock-infrastructure` references
  these images by tag and wires them together via k8s manifests (plus an
  Ingress controller for HTTPS/routing). PostgreSQL runs outside this
  deployment flow — either an in-cluster StatefulSet or an external
  managed/self-hosted instance — as its own decision when `stock-infrastructure`
  is scoped in detail; either way, `stock-backend` only depends on a
  connection string, not on how Postgres is hosted.
- **`stock-qa`**: pytest + httpx for API tests (against `stock-backend`),
  Playwright for E2E tests (against the deployed `stock-frontend`). Both
  test suites treat the sibling repos as black boxes, per the existing
  repo-relationship convention (test deployed/published outputs, not
  source trees).

## Data model

Derived from the six sheets in `Catatan_HPP_Keuangan_Bisnis.xlsx`
(Penjualan, HPP, Biaya Operasional, Stok Barang, Laba Rugi, Panduan):

- **`products`** — replaces hardcoded product rows (e.g. "Croco Nocturne
  Bag"). Fields: name, unit (pcs/roll/etc.), purchase price per unit.
- **`inventory_ledger`** — one row per stock movement (in or out), with
  product, quantity, direction, and timestamp. Current stock and any
  historical snapshot (e.g. "stock at end of March") are both derived by
  querying/summing this ledger — no separate stored "stok akhir" field.
  This replaces the spreadsheet's monthly awal/masuk/keluar/akhir columns
  with a live, queryable ledger.
- **`sales`** — revenue entries by source/product, with date and amount.
- **`expenses`** — operational cost entries (Biaya Operasional) by
  category, with date and amount.
- **`cogs_components`** — per-period COGS inputs matching the HPP sheet:
  persediaan awal, pembelian bahan baku, ongkos kirim, biaya tenaga kerja
  langsung, biaya overhead produksi, biaya kemasan, persediaan akhir.
- **Laba Rugi (P&L) is not stored.** It's computed on demand from sales,
  cogs_components, and expenses for a given period, matching the
  spreadsheet's auto-formula sheet.

## UI/UX design (web & mobile)

One responsive React app, not separate mobile/desktop codebases — a single
set of screens that adapt by viewport, since the same staff may use a phone
in the warehouse and a laptop for reporting.

- **Layout strategy**: mobile viewport is the default design target (single
  column, large touch targets, bottom tab/action bar for the most common
  actions — log sale, stock in/out). Desktop viewport (≥768px) progressively
  enhances the same screens with a persistent sidebar nav and denser tables,
  rather than a different layout tree.
- **Navigation**: bottom nav bar on mobile (Dashboard, Sales, Stock,
  Expenses, Reports); collapses into a left sidebar on desktop. Avoids
  hidden hamburger-only nav for the handful of core sections staff use daily.
- **Data entry screens** (sales, stock in/out, expenses): single-column
  forms, numeric keypad input types for quantities/amounts, large tap
  targets (≥44px), minimal required fields per screen — optimized for quick
  one-handed entry on the warehouse floor.
- **Reporting screens** (P&L, inventory value, stock ledger): tables that
  collapse into stacked cards on mobile (one card per row) and render as
  full tables on desktop, rather than forcing horizontal scroll on a phone.
- **Component approach**: Tailwind CSS with a small shared component set
  (buttons, inputs, cards, tables-that-become-cards) built once and reused
  across screens, so responsiveness is handled at the component level
  instead of per-page.
- **Accessibility baseline**: sufficient color contrast, form labels tied to
  inputs, focus states visible — standard practice, not a separate a11y
  workstream.

## Auth & access

Username/password login, JWT-based sessions. Single role tier at launch —
any authenticated account (owner + staff) can read/write. Schema/design
leaves room to add role distinctions (e.g. "warehouse" vs "owner") later
without restructuring existing tables.

## Error handling

- **Backend**: consistent JSON error shape (`{"detail": "..."}` via
  FastAPI defaults). Pydantic validates input; business-rule validation
  (e.g. reject a stock-out movement that would push stock below zero,
  reject future-dated entries) happens in the service layer.
- **Frontend**: inline form validation for entry errors, toast
  notifications for network/API failures. No optimistic UI — simple
  loading/error states are sufficient given the always-online assumption.

## Testing strategy

- **`stock-backend`**: pytest + httpx test client. Focus on business logic
  most likely to be subtly wrong — COGS calculation, running stock
  balance, P&L aggregation across periods.
- **`stock-frontend`**: Vitest + Testing Library component tests for forms
  and any client-side calculations/derived display values.
- **`stock-qa`**: Playwright E2E tests against a deployed staging
  environment, covering core flows — log a sale, log stock in/out, view
  P&L for a period.

## Out of scope (for this design)

- Multi-tenant/SaaS support.
- Offline support / local-first sync.
- Granular role-based access control (single role tier only, for now).
- Data import/migration tooling from the existing spreadsheet (can be a
  follow-up task once the schema is implemented).
