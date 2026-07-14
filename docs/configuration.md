# Configuration (.env)

BrandMonitorAI loads most settings from a **repo-root `.env`**. Docker Compose passes this file into backend/Celery containers; Next.js also reads `NEXT_PUBLIC_*` at **build time**.

Never commit real secrets. Start from [`.env.example`](../.env.example).

---

## Core application

| Variable | Required | Default / notes |
| :--- | :---: | :--- |
| `JWT_SECRET` | Prod | Signs Next.js access/refresh JWTs |
| `NEXTAUTH_SECRET` | Prod | NextAuth session signing |
| `NEXTAUTH_URL` | Yes | Public origin users open (e.g. `https://brandmonitor.zeroshield.ai`) |
| `NEXT_PUBLIC_APP_URL` | Yes | Same public origin for client + redirects |
| `MONGODB_URI` | Yes | Override in Compose: `mongodb://mongodb:27017/brandmonitorai` |
| `ALLOW_AUTH_MEMORY_FALLBACK` | No | `true` only for local demo without Mongo; **false in prod** |
| `ALLOW_MISSING_ORG` | No | Set `false` in prod for strict tenant checks |

---

## Same-origin API (recommended)

| Variable | Guidance |
| :--- | :--- |
| `NEXT_PUBLIC_USE_SAME_ORIGIN_API` | Prefer `true` — browser calls `/api/v1` on the app host |
| `NEXT_PUBLIC_MONITOR_API_URL` / `NEXT_PUBLIC_API_URL` | Leave **unset** unless you intentionally split API host |
| `INTERNAL_API_REWRITE_TARGET` | Build-time FastAPI target for Next rewrites (Compose default `http://backend:8000`) |

Production deployment pattern:

1. Browser → `https://brandmonitor.zeroshield.ai/api/v1/*`
2. Next.js rewrites → FastAPI inside the Docker network
3. Rebuild frontend after any `NEXT_PUBLIC_*` change

---

## AI / chat (Bedrock)

| Variable | Notes |
| :--- | :--- |
| `OPENAI_API_KEY` | Bearer for Bedrock OpenAI-compatible endpoint (or native IAM for other paths) |
| `OPENAI_BASE_URL` | e.g. `https://bedrock-mantle.us-east-1.api.aws/v1` |
| `CHAT_MODEL` | Model / inference profile ID |
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_REGION` | Optional IAM for Bedrock SDK |
| `AWS_BEDROCK_ENABLED` | `true` by default in Compose |

Also see [`bedrock.env.example`](../bedrock.env.example).

---

## Brand sentiment & OSINT

| Variable | Module |
| :--- | :--- |
| `GOOGLE_API_KEY` + `GOOGLE_CSE_ID` | Brand sentiment / search |
| `NEWS_API_KEY` | News sentiment |
| `YOUTUBE_API_KEY` | Video sentiment |
| `TWITTER_BEARER_TOKEN` | Social media |
| `GITHUB_TOKEN` | Repo-related scans |

---

## Data leaks

| Variable | Notes |
| :--- | :--- |
| `XPOSEDORNOT_API_KEY` | Optional paid XON features |
| `BREACH_DB_DIR` | Default `/breach-data` in Docker |
| `MONITORED_EMAILS` / `MONITORED_DOMAINS` | Beat schedules |
| `DATA_LEAKS_FREE_TIER_ONLY` | Skip paid key requirement |
| `TRUFFLEHOG_PATH` | Optional secret scanning binary |

---

## Takedown & phishing enrichment

| Variable | Notes |
| :--- | :--- |
| `URLSCAN_API_KEY` | urlscan.io |
| `VIRUSTOTAL_API_KEY` | VirusTotal |
| `GOOGLE_SAFEBROWSING_KEY` | Safe Browsing |
| `CERTSTREAM_URL` | Certificate transparency stream |
| `TAKEDOWN_RSS_URLS` | Comma-separated RSS feeds |
| `TAKEDOWN_REQUIRE_HUMAN_APPROVAL` | Gate workflow |

---

## Dark web

| Variable | Notes |
| :--- | :--- |
| `INTELX_API_KEY` | Intelligence X (if configured) |
| `TELEGRAM_API_ID` / `TELEGRAM_API_HASH` / `TELEGRAM_PHONE` | Telegram sources |
| `USE_TOR` / `TOR_PROXY_HOST` / `TOR_PROXY_PORT` | Tor sidecar |
| `DW_MONITOR_DEDUP_ENABLED` | Dedup findings before API (`false` shows all) |

---

## Alerts

| Variable | Notes |
| :--- | :--- |
| `ALERT_WEBHOOK_URL` / `SLACK_WEBHOOK_URL` | Webhook channels |
| `SMTP_HOST` + `ALERT_EMAIL_TO` | Email alerts |

---

## OAuth (Google / GitHub)

Register callbacks for both local and production:

```text
https://brandmonitor.zeroshield.ai/api/auth/callback/github
http://localhost:9002/api/auth/callback/github
https://brandmonitor.zeroshield.ai/api/auth/callback/google
http://localhost:9002/api/auth/callback/google
```

Variables: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`.

---

## Verify configuration

```bash
curl -s http://localhost:9002/api/v1/health/integrations | jq '.enabled, .disabled'
curl -s http://localhost:9002/api/v1/health/ready | jq
```

Amber banners in the UI list the exact missing `env_var` name.
