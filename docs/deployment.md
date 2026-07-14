# Deployment

## Reference production

| Item | Value |
| :--- | :--- |
| Public URL | https://brandmonitor.zeroshield.ai |
| Typical host | AWS EC2 + Docker Compose |
| Same-origin API | Browser → Next → FastAPI rewrite |

## Pre-deploy checklist

* [ ] `JWT_SECRET` and `NEXTAUTH_SECRET` are strong and unique
* [ ] `NEXTAUTH_URL` / `NEXT_PUBLIC_APP_URL` match the public HTTPS origin
* [ ] `ALLOW_AUTH_MEMORY_FALLBACK=false`
* [ ] `ALLOW_MISSING_ORG=false` when multi-tenant enforcement is required
* [ ] OAuth callback URLs registered for the public domain
* [ ] TLS terminated at load balancer / reverse proxy → Next on 9002
* [ ] `.env` present on the host (not in git)
* [ ] Images built or pulled; `docker compose up -d --build`
* [ ] `/api/v1/health/ready` returns ready / acceptable degradation
* [ ] Monitoring agents (`cadvisor`, `node-exporter`) if required by ops policy

## Deploy steps (Compose on EC2)

```bash
ssh ubuntu@<host>
cd /path/to/BrandMonitorAI
git pull
cp .env.example .env   # only first time — then edit secrets
docker compose up -d --build
docker compose ps
curl -s http://127.0.0.1:9002/api/v1/health/ready
```

Put a reverse proxy (nginx/ALB) in front of port **9002** with HTTPS.

## Environment rebuild note

Any change to `NEXT_PUBLIC_*` requires rebuilding the **frontend** image:

```bash
docker compose up -d --build frontend
```

## Rollbacks

```bash
git checkout <previous-sha>
docker compose up -d --build
```

Or repoint Compose `image:` tags to a known GHCR digest.

## Observability

* Container healthchecks in Compose
* `GET /api/v1/health/ready` and `/api/v1/health/integrations`
* Optional Kibana for CTI embeds
* Host metrics via node-exporter / cAdvisor when installed

## Security notes

* Bind data stores to `127.0.0.1` in Compose (already the default for most ports)
* Restrict SSH and security groups
* Rotate API keys periodically; never log secret values
* Prefer IAM roles on EC2 for Bedrock when possible instead of long-lived keys in `.env`

See also [runbook.md](runbook.md) for post-deploy incident response.
