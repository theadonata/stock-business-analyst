# Security review — before pushing to GitHub

Manual review across all five `stock-*` repos, done ahead of an initial
GitHub push. `/security-review`'s automated skill couldn't run here (it
needs a single git repo as its target; this is five independent repos), so
this was a direct manual pass instead: git history, secrets, auth/authz,
injection surfaces, container hardening, and dependency posture.

**Bottom line: nothing found that should block pushing.** A handful of
pre-production hardening items are listed below — none are exploitable in
the current local-dev-only setup, but should be addressed before any real
deployment (tracked here rather than in `questions.md` since they're
findings, not open questions).

## Git history / secrets

- ✅ **No secrets in git history, in any repo.** Checked `git log --all -p`
  for every historically-committed `.env*` file across all five repos —
  only `.env.example` was ever committed, and its content was always just
  an empty `GITHUB_PERSONAL_ACCESS_TOKEN=` placeholder. `.env.local` (the
  file that actually holds config, including the one that briefly had a
  real-looking GitHub token in it — see `questions.md`) was **never**
  committed in any repo; it's gitignored everywhere and stayed that way.
- ✅ **`.gitignore` correctly excludes** `.env`, `.env.local`, and
  `.claude/settings.local.json` in all five repos.
- ✅ **`.mcp.json`** (identical across all five repos) uses
  `${GITHUB_PERSONAL_ACCESS_TOKEN}` env var interpolation, not a hardcoded
  token.
- ✅ The previously-flagged possible leaked GitHub token in
  `stock-backend/.env.local` is now empty — resolved (see `questions.md`).

## Backend (`stock-backend`)

- ✅ **No SQL injection surface** — every query goes through SQLAlchemy's
  ORM query builder (`db.query(...).filter(...)`); no raw SQL / string-built
  queries anywhere in `app/`.
- ✅ **Passwords hashed with bcrypt** (via passlib), never stored plaintext.
- ✅ **JWT decode explicitly pins the algorithm** (`algorithms=[settings.JWT_ALGORITHM]`
  in `decode_access_token`), which closes off the classic "alg: none" /
  algorithm-confusion class of JWT attacks.
- ✅ **Every data endpoint requires auth** — all routers except `/auth/login`
  itself carry `dependencies=[Depends(get_current_user)]`.
- ✅ **Container runs as a non-root user** (`USER appuser` in the Dockerfile).
- ⚠️ **`JWT_SECRET` and `SEED_ADMIN_PASSWORD` ship with obviously-fake
  placeholder defaults** (`CHANGE_ME_INSECURE_DEV_ONLY_SECRET`,
  `changeme123`) in `.env.local`. This is intentional and fine for local
  dev (already commented as such in the code), but **must be overridden
  with real values before any real deployment** — `stock-infrastructure`
  should own this when it's built out.
- ⚠️ **`CORS_ORIGINS` defaults to `*`.** Also intentional for local dev
  (already commented in code), must be locked down to the real frontend
  origin before deployment.
- ⚠️ **No rate limiting on `/auth/login`.** A single-tenant internal tool
  with a handful of users is low-risk, but there's currently nothing
  stopping unlimited password-guessing attempts. Worth adding (e.g.
  `slowapi`) before exposing this outside a trusted network.
- ⚠️ **Backend dependencies are unpinned** (`fastapi`, `sqlalchemy`,
  `alembic`, etc. installed without version pins in the Dockerfile, except
  `bcrypt<4.0.0`). Fine for a fast-moving local-dev phase, but means builds
  aren't reproducible and could silently pick up a newer (possibly
  vulnerable or breaking) version later. Worth pinning exact versions
  before the app stabilizes.
- ℹ️ Postgres's port (`5433`) is exposed to the host in `docker-compose.yml`
  — correct for local dev (DB GUI access), just flagging that this
  shouldn't carry over to a production compose/manifest as-is.

## Frontend (`stock-frontend`)

- ✅ **No `dangerouslySetInnerHTML`, `eval`, or `new Function`** anywhere in
  `src/` — no obvious XSS injection surface from the code itself.
- ✅ React's default JSX escaping handles user-supplied text (product names,
  sale sources, expense categories) safely.
- ⚠️ **JWT stored in `localStorage`** (`src/api/client.ts`), not an
  httpOnly cookie. This is a common and reasonable trade-off for a JWT
  Authorization-header API (matches the spec's chosen auth design), but it
  does mean a successful XSS anywhere in the app could exfiltrate the
  token, since there's no other injection surface currently found. Worth
  knowing as a trade-off, not something to fix reactively.
- ℹ️ nginx's SPA config (`nginx.conf`) doesn't set security headers
  (`X-Content-Type-Options`, `X-Frame-Options`/`frame-ancestors`,
  `Referrer-Policy`, a CSP). Not urgent for an internal tool behind auth,
  but cheap to add before any public-facing deployment.

## Other repos

- **`stock-infrastructure`**: no application code yet — nothing to review.
- **`stock-qa`**: test-only code, talks to the other repos strictly over
  HTTP as documented; nothing sensitive.
- **`stock-business-analyst`**: specs, docs, and the original spreadsheet —
  no code execution surface.

## Recommended before any real (non-local) deployment

1. Set real `JWT_SECRET` and `SEED_ADMIN_PASSWORD` values, out of band from
   the repo (e.g. via `stock-infrastructure`'s secret management once built).
2. Lock `CORS_ORIGINS` down to the actual frontend origin.
3. Add basic rate limiting to `/auth/login`.
4. Pin backend dependency versions.
5. Add standard security response headers to the frontend's nginx config.

None of these block pushing the current code to GitHub — they're
deployment-readiness items, not vulnerabilities in what's committed today.
