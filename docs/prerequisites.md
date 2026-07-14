# Prerequisites

Before installing BrandMonitorAI, ensure the following are available.

## Required

| Tool | Version | Purpose |
| :--- | :--- | :--- |
| Git | 2.x+ | Source control |
| Docker Engine + Compose v2 | Latest stable | Recommended full-stack runtime |
| **or** Node.js | 20 LTS+ | Local UI development |
| **or** Python | 3.11+ | Local API / Celery development |

## Recommended for full Compose

| Resource | Guidance |
| :--- | :--- |
| RAM | **8 GB+** when starting Elasticsearch + Kibana + all sidecars |
| Disk | **20 GB+** free for images and volumes |
| CPU | 4+ cores preferred for Celery + ES |

On smaller machines, start a **subset** of services (see [docker.md](docker.md)):

```bash
docker compose up -d mongodb postgres redis backend frontend celery-worker celery-beat
```

## Optional tooling

| Tool / key | Enables |
| :--- | :--- |
| AWS Bedrock credentials | Primary AI chat |
| Ollama | Chat fallback without Bedrock |
| Google CSE + News + YouTube keys | Brand sentiment |
| VirusTotal / urlscan / Safe Browsing | Takedown enrichment |
| IntelX / Telegram / Tor | Dark-web depth |
| XposedOrNot API key | Paid data-leak features |

Missing optional keys do **not** block boot — modules show amber banners via `/api/v1/health/integrations`.

## Network ports (defaults)

| Port | Service |
| ---: | :--- |
| 9002 | Next.js frontend (public in Compose) |
| 8000 | FastAPI (bound to `127.0.0.1` in Compose) |
| 27016 | MongoDB host mapping |
| 5434 | PostgreSQL host mapping |
| 6380 | Redis host mapping |
| 5601 | Kibana (optional) |
| 9201 | Elasticsearch (optional) |

Confirm these ports are free on the host before `docker compose up`.

## Operating systems

* **Linux / macOS** — full feature set
* **Windows** — supported with known degradations:
  * Celery should use `--pool=solo` when run natively (Compose prefork is for Linux containers)
  * `masscan` may fall back to socket scan without admin
  * Dark-web `.onion` sources require the Tor container
