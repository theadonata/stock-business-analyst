# Open questions — review when convenient

Generated while implementing the app from
`docs/superpowers/specs/2026-08-12-stack-architecture-design.md`. None of
these blocked implementation — reasonable defaults were chosen and are
noted below each question. Flag any you want changed.

## Security

Full pre-push security review is in `findings.md` (2026-08-12) — nothing
found that blocks pushing to GitHub; a few deployment-hardening items are
listed there for before any real (non-local) deployment.

## Scope decisions made without asking

- **stock-infrastructure**: only the README was updated this pass (with a
  description + note that k8s manifests are future work). No actual
  Kubernetes manifests (Deployments/Services/Ingress) were written yet,
  since that's a sizeable scope on its own. Let me know if you want those
  built out next.
- **stock-qa**: test framework was scaffolded (pytest + httpx config,
  Playwright config) with one smoke test each, not a full test suite
  covering every flow from the spec. Let me know if you want the fuller
  suite (log a sale, stock in/out, view P&L, etc.) built out next.
- **Initial admin account**: seeded via a script with placeholder
  credentials (documented in `stock-backend/README.md`) rather than adding
  a public self-registration endpoint. Change the seeded password after
  first login. Flag if you'd rather have an open `/register` endpoint
  instead.
- **Local dev database naming**: the local Postgres container/database was
  named `stock_hpp` (not a generic `postgres`/`db`) specifically so it's
  easy to identify and won't collide with other projects' local databases
  on the same machine.

## Resolved

- **Currency formatting**: confirmed — amounts are formatted as Indonesian
  Rupiah in the UI. Already implemented (`stock-frontend/src/lib/format.ts`
  `formatCurrency`, used across Sales/Expenses/Reports), no change needed.
- **Possible leaked credential**: `stock-backend/.env.local` previously
  contained what looked like a live GitHub personal access token. As of the
  2026-08-12 security review (`findings.md`), that field is empty, and it
  was never committed to git history in any repo — resolved, no action
  needed.

## Not yet answered — needs your input

- Do you want CI (GitHub Actions or similar) wired up in
  `stock-infrastructure` or per-repo now, or is that a later pass?
  (Update: this has since been done — see
  `stock-infrastructure/docs/adr/0002-gitops-deployment-architecture.md`.)
- ~~From the 2026-08-12 Kubernetes deployability/scalability assessment
  (`stock-infrastructure/infrastructure.md`), three open infra decisions~~
  — **resolved**, see
  `stock-infrastructure/docs/adr/0002-gitops-deployment-architecture.md`:
  k3s was chosen as the k8s distribution; Postgres runs as an in-cluster
  StatefulSet using `stock-backend`'s own custom image; the frontend's
  backend-URL binding was fixed with a runtime env-injection mechanism
  (nginx `envsubst` + a fetched `/config.js`), not a separate build per
  environment. `stock-infrastructure/infrastructure.md` itself has since
  been deleted — its content is distilled into that ADR.
