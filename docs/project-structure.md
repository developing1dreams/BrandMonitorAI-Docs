# Project Structure

```text
BrandMonitorAI/
├── src/                              # Next.js application
│   ├── app/                          # App Router pages & route handlers
│   │   ├── api/                      # BFF: auth, librechat, admin helpers
│   │   ├── dashboard/
│   │   ├── *-monitoring/             # Feature module pages
│   │   ├── cti-dashboard/
│   │   ├── brand-sentiment/
│   │   └── admin/
│   ├── components/                   # Shared UI (layout, charts, chat, ui/)
│   ├── lib/                          # jwt, publicUrls, pdf, kibana helpers
│   └── middleware.ts                 # Auth gate for protected routes
├── orchestration-backend/
│   └── api/
│       ├── main.py                   # FastAPI entry + router mount
│       ├── routers/                  # Domain HTTP APIs
│       ├── services/                 # Business logic & integrations
│       ├── database/                 # Mongo / Postgres accessors
│       ├── celery_app.py
│       ├── tasks.py
│       ├── tests/
│       ├── Dockerfile
│       └── requirements*.txt
├── docs/                             # This documentation set
├── kibana/                           # kibana.yml + CTI/DNS seed assets
├── scripts/                          # seed_demo, setup-kibana-*, helpers
├── breach-data/                      # Optional local breach combo mount
├── spiderfoot/                       # SpiderFoot-related assets
├── Static Assets/                    # ZeroShield branding for GitHub
├── public/                           # Static frontend assets
├── .github/workflows/                # CI (lint, typecheck, tests, build)
├── docker-compose.yml
├── Dockerfile.frontend
├── .env.example
├── package.json
├── LICENSE
└── README.md
```

## Where to add a feature

| Change type | Primary location |
| :--- | :--- |
| New UI page | `src/app/<module>/page.tsx` + nav in `NavMenu.tsx` |
| New orchestration API | `orchestration-backend/api/routers/` + register in `main.py` |
| Scan / scheduled job | `tasks.py` + `celery_app.py` beat schedule |
| Integration client | `orchestration-backend/api/services/` |
| Auth / Next-only logic | `src/app/api/` |

See also [architecture.md](architecture.md) product pillars.
