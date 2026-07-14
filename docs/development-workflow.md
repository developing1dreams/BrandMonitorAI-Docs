# Development Workflow

## Branching

1. Create a feature branch from `main` (or your team’s default)
2. Keep PRs focused (one module or one cross-cutting concern)
3. Update docs / `docs/module-status.md` when you verify behavior

## Local loop

```bash
# Terminal A — dependencies via Compose
docker compose up -d mongodb postgres redis

# Terminal B — API
cd orchestration-backend/api && source venv/bin/activate
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Terminal C — UI
npm run dev
```

Or iterate entirely inside Compose with bind mounts via an override file.

## Quality gates (run before PR)

```bash
npm run lint
npm run typecheck
npm test
npm run build

cd orchestration-backend/api
pytest -q
```

CI (`.github/workflows/ci.yml`) runs lint, typecheck, Vitest, pytest, and `next build`.

## Coding conventions

* **Same-origin API** — UI should call `/api/v1/...` unless intentionally split
* **Typed integration errors** — prefer structured 503/429/422 over bare 500 for missing upstreams
* **Tenant filter** — Mongo-backed routers should respect `org_id` (`dependencies.py`)
* **Async scans** — long work goes to Celery; return a job id
* **Secrets** — never commit `.env`; document new vars in `.env.example` + `docs/configuration.md`

## Commit messages

Prefer clear, imperative summaries:

```text
fix(data-leaks): return 503 when XON upstream times out
docs: add Docker subset guidance for low-RAM hosts
```

## Review checklist

* [ ] UI path works when logged in
* [ ] Integration missing → amber banner / structured error
* [ ] Tests added or updated for new routers/helpers
* [ ] Docs touched if operator-facing behavior changed
