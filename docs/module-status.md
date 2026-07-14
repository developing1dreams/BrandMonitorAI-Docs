# BrandMonitorAI — Module Status Sheet

> **Living document.** Update as you verify each module. One row per top-level feature module.
> Status legend: `green` = works end-to-end with a seeded tenant, `amber` = works with missing upstream integration (graceful degradation), `red` = broken or returns 5xx.

Last verified: _pending Phase 0 smoke run on local machine_.

## How to smoke-test each module

For each row below, after `docker compose up -d --build`:

1. Open `http://localhost:9002/<ui_path>`.
2. Fill in the minimum inputs (see row).
3. Click the primary action button.
4. Note: status badge, network tab response, any console error, any red 500 banner.
5. Update the `status` column and paste evidence in the `notes` column.
6. If `red`, link to the GitHub issue you opened.

Also run:

```bash
curl http://localhost:9002/api/v1/health/integrations | jq
curl http://localhost:9002/api/v1/health/ready       | jq
```

and record which integrations are `enabled: false` in your local `.env` — that determines which modules will be `amber`.

## Module matrix

| # | Module | UI path | Backend router | Primary endpoint | Upstream keys needed | Status | Notes |
|---|--------|---------|----------------|------------------|----------------------|--------|-------|
| 1 | Dashboard (overview) | `/dashboard` | `routers/dashboard.py` | `GET /api/v1/dashboard/overview` | — | _pending_ | Newly registered in Phase 0; verify cards populate |
| 2 | Chat (AI assistant) | `/` | `routers/ai.py`, `routers/librechat.py`, Next `/api/librechat/[...libre]` | `POST /api/v1/ai/chat` | `AWS_*` or `OLLAMA_URL` | _pending_ | Amber if Bedrock unset and Ollama missing |
| 3 | Active / Passive Monitoring | `/active-passive-monitoring` | `routers/monitoring.py` | `POST /api/v1/monitor/start` | `NUCLEI_PATH`, `ZAP_API_URL`, `RENGINE_*` | _pending_ | Polls every 5s — this is "live monitoring" |
| 4 | External Surface (ASM) | `/external-surface-monitoring` | `routers/external_surface.py` | `POST /api/v1/external-surface/scan` | SpiderFoot container | _pending_ | Falls back to Layer 1 only without SpiderFoot |
| 5 | Data Leaks | `/data-leak-monitoring` | `routers/data_leaks.py` | `GET /api/v1/data-leaks/summary` | `XPOSEDORNOT_API_KEY` (paid), `SHODAN_API_KEY` | _pending_ | Stats must be tenant-scoped (Phase 1.6) |
| 6 | Dark Web | `/dark-web-monitoring` | `routers/dark_web.py` | `POST /api/v1/dark-web/scan` | `INTELX_API_KEY`, `TELEGRAM_*`, Tor | _pending_ | 500 → 503 conversion is Phase 1.2 |
| 7 | Takedown | `/takedown-monitoring` | `routers/takedown.py` | `POST /api/v1/takedown/request` | `URLSCAN_API_KEY`, `VIRUSTOTAL_API_KEY`, `GOOGLE_SAFEBROWSING_KEY` | _pending_ | |
| 8 | Phishing Detection | `/phishing-detection` | `routers/phishing.py` | `POST /api/v1/phishing/scan` | `OPENAI_API_KEY` | _pending_ | |
| 9 | App Store | `/app-store-monitoring` | `routers/app_store.py` | `POST /api/v1/app-store/search` | — (public scraping) | _pending_ | Audit baseline says operational |
| 10 | DMARC | `/dmarc-monitoring` | `routers/dmarc.py` | `POST /api/v1/dmarc/analyze` | — (DNS only) | _pending_ | Audit baseline says operational |
| 11 | DNS | `/dns-monitoring` | `routers/dns.py` | `POST /api/v1/dns/monitor` | — (`dnspython`) | _pending_ | Empty results possible if `dnspython` missing in image |
| 12 | Social Media | `/social-media-monitoring` | `routers/social_media.py` | `POST /api/v1/social-media/scan` | Twitter/X API, Telethon, Instaloader | _pending_ | |
| 13 | Brand Sentiment | `/brand-sentiment` | `routers/brand_sentiment.py` | `POST /api/v1/brand-sentiment/analyze` | `GOOGLE_API_KEY`+`GOOGLE_CSE_ID`, `NEWS_API_KEY`, `YOUTUBE_API_KEY` | _pending_ | Mongo cache fallback in Phase 1.7 |
| 14 | CTI Dashboard | `/cti-dashboard` | `routers/cti.py` | `GET /api/v1/cti/overview` | Kibana/Elasticsearch | _pending_ | Many sub-pages — smoke test the landing only |
| 15 | Policy Scan | (backend only) | `routers/policy_scan.py` | `POST /api/v1/policy-scan/run` | `checkov`, `prowler` binaries | _pending_ | Newly registered in Phase 0 |
| 16 | SAST | `/sast` | **none — no backend router** | n/a | n/a | **red** | Open question for senior: build router or remove page |
| 17 | Admin — Users | `/admin/users` | `routers/auth.py` (+ `/api/admin/users/*` Next) | `GET /api/admin/users` | — | _pending_ | |
| 18 | Admin — Analytics | `/admin/analytics` | — | — | — | _pending_ | |

## Cross-cutting checks

| Area | Check | Pass? | Evidence |
|------|-------|-------|----------|
| Auth — middleware protects routes | `curl -i http://localhost:9002/dashboard` without cookie → 307 to `/login` | _pending_ | Phase 0 just added |
| Auth — login rejects empty fields | `POST /api/auth/login {}` → 400 | _pending_ | Already present |
| Auth — memory fallback gated in prod | `ALLOW_AUTH_MEMORY_FALLBACK=false` blocks login when Mongo down | _pending_ | Phase 3.6 |
| Backend — no red 500 in UI | Trigger every "amber" module → expect typed 503 with `error.detail.error` | _pending_ | Phase 1.2 |
| Backend — `/api/v1/health/integrations` | Returns enabled/disabled matrix | _pending_ | Phase 0 just added |
| Tenant — isolation | Seed two tenants, scan in A, list in B → empty | _pending_ | Phase 4.2 |
| CI — green on main | Lint + typecheck + pytest + vitest + `next build` | _pending_ | Phase 4.5 |

## Known degraded on Windows (don't try to fix)

- `masscan` requires admin → silently falls back to socket scan (less accurate)
- Celery **must** use `--pool=solo` (no `fork()` on Windows)
- `.onion` sources need Tor container running
- SpiderFoot layers 2–4 require the SpiderFoot docker service

## Template for a "module broken" report

```
Module: <name>
UI path: <path>
Backend endpoint: <METHOD PATH>
Steps: 1) … 2) …
Expected: <verbatim>
Actual: <verbatim + status code>
Screenshot: <link>
Fix PR: #<number>
Phase: <0-5>
```
