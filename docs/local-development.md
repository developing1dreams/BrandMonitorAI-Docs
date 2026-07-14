# Local Development

Use this path when iterating on UI or API without rebuilding every Compose service.

## Frontend only

```bash
npm install
cp .env.example .env.local
# Point API to a running backend, or rely on Next rewrites if backend is on :8000

npm run dev
# → http://localhost:9002
```

Scripts (`package.json`):

| Script | Purpose |
| :--- | :--- |
| `npm run dev` | Turbopack dev server on port **9002** |
| `npm run build` | Production build |
| `npm run lint` | ESLint |
| `npm run typecheck` | `tsc --noEmit` |
| `npm test` | Vitest |

## Backend + Celery

```bash
cd orchestration-backend/api
python -m venv venv
# Windows: venv\Scripts\activate
source venv/bin/activate
pip install -r requirements.txt
pip install -r requirements-dev.txt   # pytest, etc.

# Ensure Mongo / Postgres / Redis are reachable (Compose deps or local installs)
python main.py
# Uvicorn on :8000

# Separate terminal — Windows uses solo pool:
celery -A celery_app worker --loglevel=info --pool=solo
celery -A celery_app beat --loglevel=info
```

## Convenience scripts

| Script | Purpose |
| :--- | :--- |
| `start-platform.bat` | Windows: Redis container + backend + Celery + frontend |
| `start-platform-no-docker.bat` | Backend + frontend only |
| `open-platform-in-browser.bat` | Open local UI |

## Same-origin rule

During local full-stack runs, prefer:

* UI on `http://localhost:9002`
* Browser fetches `/api/v1/...` (Next rewrite → FastAPI)
* Do **not** set `NEXT_PUBLIC_API_URL` unless you intentionally split hosts

## Hot-reload tips

* Next.js: App Router HMR via `npm run dev`
* FastAPI: run uvicorn with `--reload` for router work (dev only)
* Rebuild Compose backend image after `requirements.txt` or Dockerfile tool changes

## Seeding

```bash
python scripts/seed_demo.py --brand acme --tenant demo
```
