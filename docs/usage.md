# Usage Guide

Operator-oriented walkthrough of BrandMonitorAI modules. UI paths assume a logged-in session at the app origin (local `http://localhost:9002` or [https://brandmonitor.zeroshield.ai](https://brandmonitor.zeroshield.ai)).

## Sign in

1. Open `/login`
2. Use credentials or OAuth (Google / GitHub) if configured
3. Protected routes redirect to `/login?next=…` when unauthenticated

Change password via the in-app flow (`/api/auth/change-password`) — rotating password invalidates older JWTs.

## Dashboard

* **Path:** `/dashboard`
* Overview cards for scans, alerts, and recent activity
* Use as the daily landing page for SOC / brand ops

## Chat assistant

* **Path:** `/` (Chat in nav)
* Asks product-scoped questions (modules, workflows)
* Requires Bedrock (or Ollama fallback) — see [configuration.md](configuration.md)

## External Surface (ASM)

1. Open `/external-surface-monitoring`
2. Enter a domain / target
3. Start scan → poll until complete
4. Export CSV/JSON as needed

Requires SpiderFoot container for deeper layers; Layer 1 may still run without it.

## Data Leak Monitoring

1. Open `/data-leak-monitoring`
2. Configure monitored emails/domains or run checks
3. Review summary widgets (`org_id` scoped when multi-tenant)
4. Optional: mount `./breach-data` for local combo search

## Dark Web Monitoring

1. Open `/dark-web-monitoring`
2. Run brand monitor / inspect source health
3. Triage findings in review queue → promote to cases
4. Celery Beat schedules recurring jobs automatically when the stack is up

Tor improves `.onion` coverage.

## Takedown Monitoring

1. Open `/takedown-monitoring`
2. Create or review threats
3. Enrichment uses urlscan / VirusTotal / Safe Browsing when keys are set
4. Generate reports and track request status

## Phishing Detection

1. Open `/phishing-detection`
2. Submit URL or batch inputs
3. Review scores / hunt summaries
4. Optional LLM validation when AI keys are present

## App Store Monitoring

1. Open `/app-store-monitoring`
2. Search stores for brand / lookalikes
3. Track impersonation threats over time

## DNS & DMARC

| Module | Path | Use |
| :--- | :--- | :--- |
| DNS | `/dns-monitoring` | Monitor DNS posture / records |
| DMARC | `/dmarc-monitoring` | Analyze policy and generate reports |

## Active / Passive Monitoring

1. Open `/active-passive-monitoring`
2. Start passive and/or active scan jobs
3. Poll status; export results

Uses recon and port-scan tooling with safe fallbacks when privileged scanners are unavailable.

## Social Media & Brand Sentiment

| Module | Path |
| :--- | :--- |
| Social Media | `/social-media-monitoring` |
| Brand Sentiment | `/brand-sentiment` |

Sentiment benefits from Google CSE, News, and YouTube API keys. PDF export is available for reports where wired.

## CTI Dashboard

* **Path:** `/cti-dashboard`
* Drill into coverage and related CTI views
* Optional Kibana embeds via `NEXT_PUBLIC_KIBANA_*` URLs

## Admin

* `/admin/users`, `/admin/analytics` — tenant/user administration (authorized roles)

## Integration banners

If a module shows an amber banner:

1. Note the missing `env_var`
2. Add it to `.env`
3. Restart affected containers
4. Re-check `/api/v1/health/integrations`

## Smoke checklist

```bash
curl -s http://localhost:9002/api/v1/health/ready | jq
curl -s http://localhost:9002/api/v1/health/integrations | jq
```

Track formal module verification in [module-status.md](module-status.md).
