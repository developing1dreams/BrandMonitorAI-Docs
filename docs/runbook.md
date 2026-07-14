# Operator Runbook

Fast checklists for the "demo just broke, what do I check?" moment.

Part of **BrandMonitorAI** / [ZeroShield](https://zeroshield.ai). See also [troubleshooting.md](troubleshooting.md).

## 0. First 60 seconds of any incident

```bash
curl -s http://localhost:9002/api/v1/health/ready        | jq
curl -s http://localhost:9002/api/v1/health/integrations | jq '.disabled, .enabled'
docker compose ps
docker compose logs --tail=50 backend celery-worker celery-beat
```

If `ready.components.mongodb` or `ready.components.redis` are not `ok`, jump to §2.

## 1. "My demo looks empty"

| Symptom | Most likely cause | Fix |
|---|---|---|
| Dashboard cards are all `0` | Demo data was never seeded | `python scripts/seed_demo.py --brand acme --tenant demo` |
| DNS page shows no records | `dnspython` missing in image | Rebuild `api` image after pulling (`requirements.txt` now pins `dnspython>=2.6.0`) |
| Brand Sentiment widgets blank | Missing `GOOGLE_API_KEY`, `NEWS_API_KEY`, `YOUTUBE_API_KEY` | Open `/settings` → Integrations tab; set the missing env vars and restart `api` |
| Dark Web scan returns 503 | IntelX/Tor/Telegram unreachable | 503 body now tells you which provider and which env var — set it and retry |
| Data-Leak numbers look "global" instead of tenant-specific | Not passing `org_id` | UI should call `/api/v1/data-leaks/summary?org_id=<tenant>`; check query string |
| "Critical Alerts" card doesn't match `/api/v1/alerts/stats` | CTI page cached metrics | Click Refresh (top-right of CTI dashboard) — the `criticalAlertsCount` prop re-fetches |

## 2. Dependencies down

### MongoDB

```bash
docker compose ps mongodb
docker compose logs --tail=100 mongodb
# connectivity from the API container
docker compose exec api python -c "from database.mongodb import get_mongodb_db; db = get_mongodb_db(); print('ok' if db is not None else 'none'); db.list_collection_names()"
```

If Mongo is down, `/api/auth/login` will **reject** logins in production (`ALLOW_AUTH_MEMORY_FALLBACK != "true"`) with 503. Fix Mongo before blaming the frontend.

### Redis / Celery

```bash
docker compose ps redis celery_worker celery_beat
docker compose exec celery_worker celery -A celery_app inspect ping
# Windows only:
# Celery must use --pool=solo (already set in docker-compose); do NOT change it.
```

If `celery inspect ping` fails, restart `celery_worker`. Watch for `masscan` needing admin — on Windows we silently fall back to socket scan; that's expected.

## 3. Auth

```bash
# Is route protection active?
curl -i http://localhost:9002/dashboard  # expect 307 → /login

# Is NextAuth happy?
curl -i http://localhost:9002/api/auth/session

# Change password for the current user
curl -X POST http://localhost:9002/api/auth/change-password \
  -H 'Content-Type: application/json' \
  -b cookies.txt -c cookies.txt \
  -d '{"currentPassword":"old","newPassword":"newsecret!"}'
```

After a password change, `passwordChangedAt` is written and other sessions are invalidated at next request via `isTokenStale()`.

## 4. "It was working yesterday"

1. `git log --oneline -n 20` — what merged?
2. `docker compose logs api | grep ERROR | tail -n 20`
3. `curl /api/v1/health/integrations | jq '.disabled'` — did a key expire?
4. If a single module broke, check the router in `orchestration-backend/api/routers/<module>.py` and grep for recent edits.

## 5. Integration-banner playbook (what the UI is telling you)

| `detail.error` | UI banner variant | What it means | Fix |
|---|---|---|---|
| `integration_not_configured` | amber | Backend is missing an API key | Set `detail.env_var` in `.env`, restart `api` |
| `upstream_unavailable` | orange | Network/DNS/timeout to a third-party | Retry; if persistent, check provider status page |
| `upstream_rate_limited` | yellow | We burst past the provider's free tier | Wait for `retry_after` seconds, reduce scan frequency |
| `upstream_validation_error` | blue | The provider rejected our inputs | UI must re-ask the analyst for valid inputs |

## 6. Release checklist (before you tell anyone "it's ready")

- [ ] `npm run lint && npm run typecheck && npm test`
- [ ] `cd orchestration-backend/api && pytest -q`
- [ ] `npm run build` succeeds locally
- [ ] `docker compose up -d --build` from a clean state
- [ ] `/api/v1/health/ready` returns `ready` (not `degraded`)
- [ ] `/api/v1/health/integrations` — every key you expect is `enabled: true`
- [ ] `python scripts/seed_demo.py --brand <demo-brand> --tenant <demo-tenant>` completes
- [ ] Click through all 12 modules once; note amber bannered ones in the demo script

## 7. Known Windows degradations (accept these, don't fix at OS level)

- Celery runs with `--pool=solo`.
- `masscan` requires admin → socket fallback is active.
- `.onion` sources require the Tor sidecar container.
- Spacy/torch first run may take a few minutes to download models.
