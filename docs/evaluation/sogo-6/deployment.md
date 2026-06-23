# SOGo 6 — Deployment Evaluation

## Architecture

SOGo 6 consists of three containers/services:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Frontend    │     │  Backend     │     │  Agent       │
│  Next.js     │ ←──→│  Flask       │ ←──→│  Celery      │
│  (React)     │     │  gunicorn    │     │  + Redis     │
└──────────────┘     └──────┬───────┘     └──────────────┘
                            │
                    ┌───────┴───────┐
                    │  Database     │
                    │  PostgreSQL   │
                    └───────────────┘
```

## Component Breakdown

| Component | Tech | Notes |
|---|---|---|
| Frontend server | Next.js (React) | No backend config needed, just backend URL |
| Backend server | Python / Flask / gunicorn | RESTful API |
| Agent | Celery (Python) | Distributed task queue |
| Database | PostgreSQL | Required; MariaDB supported |
| Cache | Redis | Replaces Memcached |

## Configuration Tiers

1. **Process settings** (ENV or file) — database URL, Redis URL, encryption secrets
2. **Dynamic settings** (database) — IMAP server, auth, user sources, everything else
3. **Admin UI** — Web interface for dynamic settings with import wizard

## What to Evaluate

- [ ] Docker Compose reference configuration
- [ ] Migration script reliability (SOGo 5 → SOGo 6)
- [ ] Configuration export/import for pipeline deployment
- [ ] Performance under load (single-node vs cluster)
- [ ] Redis sizing requirements
- [ ] Database migration downtime
- [ ] Backup and restore procedures
