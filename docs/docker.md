# Docker Setup

BrandMonitorAI ships a production-oriented [`docker-compose.yml`](../docker-compose.yml) that starts the full platform.

## Start / stop

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f frontend backend celery-worker
docker compose down
```

## Services overview

| Service | Container name | Role |
| :--- | :--- | :--- |
| `frontend` | `brandmonitorai-frontend` | Next.js UI (:9002) |
| `backend` | `brandmonitorai-backend` | FastAPI (:8000 → localhost) |
| `celery-worker` | `brandmonitorai-celery-worker` | Async scans |
| `celery-beat` | `brandmonitorai-celery-beat` | Schedules |
| `mongodb` | `brandmonitorai-mongodb` | Primary app data |
| `postgres` | `brandmonitorai-postgres` | Structured scan results |
| `redis` | `brandmonitorai-redis` | Cache + Celery broker |
| `elasticsearch` | `brandmonitorai-elasticsearch` | Optional analytics |
| `kibana` | `brandmonitorai-kibana` | Optional dashboards |
| `meilisearch` | `brandmonitorai-meilisearch` | Optional search |
| `spiderfoot` | `brandmonitorai-spiderfoot` | ASM |
| `owasp-zap` | `brandmonitorai-owasp-zap` | Web scanning |
| `tor` | `brandmonitorai-tor` | Dark-web proxy |
| `falco` | `falco` | Runtime monitor stub |
| `nikto` | `brandmonitorai-nikto` | On-demand scanner image |

Published monitoring agents (often added on production hosts alongside Compose): `cadvisor`, `node-exporter`.

## Images

Application images default to GHCR:

* `ghcr.io/damarasingu/brandmonitorai-frontend:latest`
* `ghcr.io/damarasingu/brandmonitorai-backend:latest`
* `ghcr.io/damarasingu/brandmonitorai-celery-worker:latest`
* `ghcr.io/damarasingu/brandmonitorai-spiderfoot:latest`

`docker compose up -d --build` rebuilds from local Dockerfiles when needed.

## Resource-friendly subset

```bash
docker compose up -d mongodb postgres redis backend frontend celery-worker celery-beat
```

Add sidecars as needed: `spiderfoot`, `tor`, `elasticsearch kibana`, `owasp-zap`.

## Override file

Copy [`docker-compose.override.example.yml`](../docker-compose.override.example.yml) to `docker-compose.override.yml` for local port or volume tweaks (gitignored pattern).

## Health checks

```bash
curl -s http://localhost:9002/api/v1/health/ready | jq
docker compose exec backend python -c "print('ok')"
docker compose exec celery-worker celery -A celery_app inspect ping
```

## Frontend bind note

The frontend command forces `HOSTNAME=0.0.0.0` so Docker’s injected hostname does not break HTTP listeners (`ERR_EMPTY_RESPONSE`).

## Cleanup

```bash
./docker-cleanup.sh          # Linux/macOS helper
# or
docker compose down -v
docker system prune -f
```
