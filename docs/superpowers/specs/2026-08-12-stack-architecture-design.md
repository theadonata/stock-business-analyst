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
- Self-hosted deployment via Docker on a VPS.
- Always-online; no offline support needed for now.

## Architecture

Two independently deployable services communicating over a versioned REST
API, matching the existing repo split:

- **`stock-backend`**: FastAPI + PostgreSQL. Exposes `/api/v1/...` JSON REST
  endpoints. OpenAPI docs auto-generated at `/docs`, serving as the API
  contract for `stock-frontend` and `stock-qa`.
- **`stock-frontend`**: React + Vite + TypeScript SPA, Tailwind CSS for
  styling, TanStack Query for API data fetching/caching. Built to static
  files, served behind the reverse proxy. Mobile-first responsive layout.
- **`stock-infrastructure`**: `docker-compose.yml` running backend,
  frontend, and PostgreSQL containers on a VPS, with Caddy as a reverse
  proxy (automatic HTTPS, minimal config).
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
