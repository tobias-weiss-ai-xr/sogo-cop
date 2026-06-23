# SOGo 6 — Feature Evaluation

## Core Features (vs SOGo 5)

| Feature | SOGo 5 | SOGo 6 | Notes |
|---|---|---|---|
| Webmail | ✓ (AngularJS) | ✓ (React/Next.js) | Complete rewrite |
| Calendar (CalDAV) | ✓ | ✓ | Advanced development as of May 2026 |
| Address Book (CardDAV) | ✓ | Not yet done | "Not the most complicated part" |
| ActiveSync | ✓ | Reconsidered | May arrive after v1 release |
| MAPI over HTTP | ✗ | Planned | Microsoft protocol replacement |
| Sieve mail filtering | ✓ | ✓ | |
| IMAP sync | ✓ | ✓ | |
| SMTP outgoing | ✓ | ✓ | |
| Push notifications | ✓ | ✓ (via Celery Agent) | Task-queue based now |
| Multi-domain | Limited | Built-in | Default + overrides per domain |
| Admin interface | sogo.conf file + CLI | Web UI + REST API | Dynamic, no restart needed |
| SSO (OIDC, SAML, CAS) | ✓ | ✓ | |
| LDAP / SQL user sources | ✓ | ✓ | |
| Jitsi integration | ✓ | ✓ | |

## New in SOGo 6

- **Modular architecture** — API → Interface → Module → Manager layers
- **RESTful API** — All admin actions available via HTTP; replace `sogo-tool` with curl
- **Celery Agent** — Task queue for scheduled sends, delayed actions, async processing
- **Redis-based caching** — Replaces Memcached; also used for task queue
- **Dynamic configuration** — Parameters stored in DB, no restart needed
- **Configuration export/import** — JSON-based for pipeline deployment
- **Container-native** — Official Docker images (backend + frontend + agent)

## Removed / Deprecated

| Feature | Status | Alternative |
|---|---|---|
| ActiveSync | Initially planned for removal, then reconsidered | MAPI over HTTP (future); may ship post-v1 |
| Memcached | Replaced | Redis |
| MySQL/MariaDB | Initially planned to drop MariaDB, then reconsidered | PostgreSQL recommended, MariaDB still supported |
| sogo.conf | Replaced | Admin UI + ENV + database |
| AngularJS frontend | Replaced | React / Next.js |
| Objective-C backend | Replaced | Python (Flask) |

## Migration Path

SOGo 6 provides:
- Database migration scripts (SOGo 5 → SOGo 6 schema)
- Configuration import (upload `sogo.conf` via admin UI)
- Side-by-side deployment possible (containers)
