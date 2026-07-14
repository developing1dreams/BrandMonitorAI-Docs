# Installation & Setup

## 1. Clone the repository

```bash
git clone https://github.com/vartulzeroshieldai/BrandMonitorAI.git
cd BrandMonitorAI
```

## 2. Create environment file

```bash
cp .env.example .env
```

Minimum variables for a local/demo run:

```env
JWT_SECRET=change-me-to-a-long-random-string
NEXTAUTH_SECRET=change-me-to-another-long-random-string
NEXTAUTH_URL=http://localhost:9002
NEXT_PUBLIC_APP_URL=http://localhost:9002
ALLOW_AUTH_MEMORY_FALLBACK=true
```

For production, set `ALLOW_AUTH_MEMORY_FALLBACK=false` and use the public HTTPS origin (e.g. `https://brandmonitor.zeroshield.ai`).

Full variable reference: [configuration.md](configuration.md).

## 3. Start with Docker Compose

```bash
docker compose up -d --build
```

First build can take several minutes (backend image installs scanner toolchains).

Verify:

```bash
docker compose ps
curl -s http://localhost:9002/api/v1/health/ready
curl -s http://localhost:9002/api/v1/health/integrations
```

## 4. Open the application

| URL | Purpose |
| :--- | :--- |
| http://localhost:9002 | Main UI |
| http://localhost:9002/login | Sign in |
| http://localhost:9002/docs | FastAPI Swagger (rewritten) |

## 5. Seed demo data (optional)

```bash
# Requires Python deps and Mongo reachable (Compose network or localhost mapping)
python scripts/seed_demo.py --brand acme --tenant demo
```

Default demo credentials (if seeded as documented in the script): check `scripts/seed_demo.py` output.

## 6. Stop / clean

```bash
docker compose down
# Remove volumes (destructive):
docker compose down -v
```

## Next steps

* [Docker details](docker.md)
* [Local development](local-development.md)
* [Usage guide](usage.md)
* [Deployment](deployment.md)
