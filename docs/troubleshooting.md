# Troubleshooting

For live incident checklists, start with [runbook.md](runbook.md).

## First 60 seconds

```bash
curl -s http://localhost:9002/api/v1/health/ready | jq
curl -s http://localhost:9002/api/v1/health/integrations | jq '.disabled, .enabled'
docker compose ps
docker compose logs --tail=80 backend celery-worker celery-beat frontend
```

## Common issues

### UI shows `ERR_EMPTY_RESPONSE` on port 9002

Frontend must listen on `0.0.0.0`. Compose command already forces `HOSTNAME=0.0.0.0`. Rebuild/restart `frontend` if you customized the command.

### Login fails / 503 on auth

* MongoDB unhealthy → fix `mongodb` container first
* Production with `ALLOW_AUTH_MEMORY_FALLBACK=false` will not silently fall back to in-memory users

```bash
docker compose ps mongodb
docker compose logs --tail=100 mongodb
```

### Modules show amber banners / empty data

Missing upstream keys. Check integrations endpoint and set the named env vars, then restart `backend` / workers.

### Celery not processing jobs

```bash
docker compose ps redis celery-worker celery-beat
docker compose exec celery-worker celery -A celery_app inspect ping
```

Restart worker if ping fails. On native Windows (non-Docker worker), use `--pool=solo`.

### DNS module returns empty

Ensure `dnspython` is installed in the API image (`requirements.txt`). Rebuild backend.

### Dark web / onion failures

Start Tor sidecar; confirm `USE_TOR=true` and proxy host/port. IntelX/Telegram keys may be required for specific sources.

### Browser calls wrong API host

Unset `NEXT_PUBLIC_API_URL` / `NEXT_PUBLIC_MONITOR_API_URL`, set `NEXT_PUBLIC_USE_SAME_ORIGIN_API=true`, rebuild frontend.

### OAuth redirect mismatch

`NEXTAUTH_URL` must equal the URL in the browser address bar. Register exact callback URLs in Google/GitHub consoles.

### Out of memory on Compose up

Do not start Elasticsearch/Kibana on low-RAM hosts. Use the subset in [docker.md](docker.md).

### Dashboard cards all zero

Seed demo data:

```bash
python scripts/seed_demo.py --brand acme --tenant demo
```

## Integration error map

| `detail.error` | Meaning | Action |
| :--- | :--- | :--- |
| `integration_not_configured` | Missing env key | Set `detail.env_var`, restart |
| `upstream_unavailable` | Network/timeout | Retry; check provider status |
| `upstream_rate_limited` | Rate limit | Back off / lower scan frequency |
| `upstream_validation_error` | Bad input | Correct analyst inputs |

## Still stuck?

1. Check recent git changes
2. Capture `/health/ready` + `/health/integrations` JSON
3. Attach `docker compose logs --tail=200` for failing services
4. Email [support@zeroshield.ai](mailto:support@zeroshield.ai)
