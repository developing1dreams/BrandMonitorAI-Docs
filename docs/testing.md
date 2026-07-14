# Testing

## Frontend (Vitest)

```bash
npm install
npm test                 # vitest run
npm run test:watch       # interactive
```

Config: `vitest.config.ts`

## Backend (pytest)

```bash
cd orchestration-backend/api
pip install -r requirements-dev.txt
pytest -q
```

Tests live under `orchestration-backend/api/tests/`.

## Manual / smoke

```bash
docker compose up -d --build
curl -s http://localhost:9002/api/v1/health/ready | jq
curl -s http://localhost:9002/api/v1/health/integrations | jq
curl -i http://localhost:9002/dashboard   # expect 307 → /login when logged out
```

Walk modules using [module-status.md](module-status.md).

## Demo seed

```bash
python scripts/seed_demo.py --brand acme --tenant demo
```

## Coverage goals

Aim for happy-path coverage on new FastAPI routers and critical frontend helpers (JWT, public URL resolution, integration banners). Expand fixtures as the suite grows.
