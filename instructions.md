# Running the Stock/HPP app locally

Step-by-step instructions to run the whole app (backend + frontend) via
Docker and access it as a human tester. Covers both repos since they're
deployed independently but need to run together for a full walkthrough.

Prerequisites: Docker + Docker Compose installed and running.

## 1. Start the backend (`stock-backend`)

```bash
cd stock-backend
docker compose up --build
```

What this does:
- Builds the FastAPI app image.
- Starts a PostgreSQL container named `stock_hpp_postgres` (database
  `stock_hpp_db`, volume `stock_hpp_pgdata` — named distinctly so it won't
  collide with other projects' local Postgres containers).
- Runs database migrations automatically (`alembic upgrade head`) —
  schema only, **no data**. The database starts empty.
- Starts the API on **http://localhost:8000**.

Wait for a line like `Application startup complete` before continuing.

### Seed a login (one-time, required)

The database has no users until you run this. Open a second terminal:

```bash
cd stock-backend
docker compose exec stock_hpp_app python -m scripts.seed_admin
```

This creates one admin account:
- **Username:** `admin`
- **Password:** `changeme123`

(These are placeholder defaults — override them by setting
`SEED_ADMIN_USERNAME` / `SEED_ADMIN_PASSWORD` before running the seed
command, or just change the password after your first login.) There is no
public self-registration page; this script is the only way an account gets
created.

### Verify the backend is up

Open **http://localhost:8000/docs** — this is the interactive Swagger UI
for the API. You can exercise any endpoint here directly, which is useful
for sanity-checking before even opening the frontend.

## 2. Start the frontend (`stock-frontend`)

In a new terminal:

```bash
cd stock-frontend
docker compose up --build
```

This builds the React app and serves it via nginx on
**http://localhost:8080**. By default it's already configured to talk to
the backend at `http://localhost:8000` (matches step 1's default port, so
no extra config is needed unless you changed something).

If your backend is running somewhere other than `http://localhost:8000`,
pass the real URL as a build arg instead:

```bash
VITE_API_BASE_URL=http://your-backend-host:8000 docker compose up --build
```

## 3. Access and test the app as a human

1. Open **http://localhost:8080** in a browser.
   - To test the mobile layout on a real phone: find your machine's LAN IP
     (e.g. `192.168.1.x`), make sure the phone is on the same network, and
     open `http://<your-lan-ip>:8080`.
2. Log in with `admin` / `changeme123`.
3. **Add at least one product first** — go to the **Products** page (in
   the nav, between Dashboard and Sales), fill in name/unit/purchase
   price, and submit. Sales and Stock movements both need an existing
   product to log against.
4. Try the core flows:
   - **Stock**: log a stock-in movement for the product you just added,
     confirm "Current stock" updates.
   - **Sales**: log a sale (revenue source, optional linked product,
     date, amount).
   - **Expenses**: log an operational expense (category, date, amount).
   - **Reports**: pick a month and view the computed profit & loss
     (Laba Rugi) breakdown — total sales, COGS, gross profit, expenses,
     net profit.
5. Try a couple of edge cases to confirm validation works:
   - Log a stock-out larger than the current stock on hand — should be
     rejected with an inline error (backend enforces "can't go below
     zero").
   - Pick a future date on any entry form — should be rejected.

## Running the automated test suites (optional, not required to use the app)

**Backend** (no Docker required — uses an in-memory SQLite DB):
```bash
cd stock-backend
pip install -e ".[dev]"
pytest
```

**Frontend**:
```bash
cd stock-frontend
npm install
npm run test
```

**QA repo** (`stock-qa`) — smoke tests against already-running instances
from steps 1–2 above:
```bash
cd stock-qa/api-tests
API_BASE_URL=http://localhost:8000 pytest

cd stock-qa/e2e
BASE_URL=http://localhost:8080 npm test
```

## Stopping / resetting

Stop everything:
```bash
# in each repo's terminal
docker compose down
```

Stop and wipe the database completely (fresh empty DB next time you run
`docker compose up`):
```bash
cd stock-backend
docker compose down -v
```

## Troubleshooting

- **Frontend loads but nothing works / network errors**: confirm the
  backend is actually running and reachable at the URL the frontend was
  built with (`VITE_API_BASE_URL`). Vite bakes this in at build time, so
  changing it requires rebuilding the frontend image, not just restarting
  the container.
- **Can't log in**: confirm you ran the seed script (step 1) — a fresh
  database has zero users by design.
- **Port already in use**: backend uses `8000` (API) and `5433` (Postgres,
  mapped to the container's `5432`); frontend uses `8080`. Stop whatever
  else is using those ports, or edit the `ports:` mapping in the relevant
  `docker-compose.yml`.
