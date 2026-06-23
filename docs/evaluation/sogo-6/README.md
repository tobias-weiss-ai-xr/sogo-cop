# SOGo 6 Evaluation

Resources for evaluating **SOGo 6** — the complete rewrite of SOGo in Python (Flask) and React/Next.js.

- **Demo:** [demov6.sogo.nu](https://demov6.sogo.nu/SOGo/)
- **Status:** Public beta planned for summer 2026
- **License:** GPL v2+

## Key Architectural Changes from SOGo 5

| Aspect | SOGo 5 | SOGo 6 |
|---|---|---|
| Backend | Objective-C | Python (Flask / gunicorn) |
| Frontend | AngularJS | React.js / Next.js |
| Architecture | Monolithic | Modular (API → Interface → Module → Manager) |
| Cache | Memcached | Redis |
| Database | MySQL/MariaDB/PostgreSQL | PostgreSQL (MariaDB also supported after community feedback) |
| Config | `sogo.conf` file | Admin UI + ENV vars + database |
| Task Queue | — | Celery + Redis (Agent) |
| ActiveSync | Supported | Reconsidered — may arrive post-v1 |
| Multi-Domain | Limited | Built-in |

## Evaluation Focus Areas

- **[Accessibility](accessibility.md)** — WCAG 2.2 AA audit of the web UI
- **[Features](features.md)** — Feature parity and new capabilities vs SOGo 5
- **[Deployment](deployment.md)** — Container architecture, migration path
- **[Performance](performance.md)** — Benchmarks, scalability, resource usage
