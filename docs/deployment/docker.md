# SOGo in Containers -- Docker / Podman / Compose

- **Last updated:** 2026-06-26
- **Scope:** SOGo 5.x (stable) and SOGo 6 (pre-beta) container deployment
- **Audience:** practitioners evaluating or running SOGo in Docker / Podman / Compose / Kubernetes
- **Status:** Reference guide -- not a substitute for a tested runbook

## TL;DR

- **For production today:** run SOGo 5.x >= 5.12.8 (security floor; see
  `../evaluation/sogo-6/known-limitations.md`). The canonical community Docker
  image is `sonroyaalmerol/docker-sogo` (built from Alinto's stable source tag).
  The `alinto/sogo` image on Docker Hub is **not** published by Alinto for
  stable releases -- production builds require a support subscription from
  [packages.sogo.nu](https://packages.sogo.nu).
- **For evaluation of the next generation:** the SOGo 6 demo is at
  [https://demov6.sogo.nu/SOGo/](https://demov6.sogo.nu/SOGo/); the public
  container image is **not yet published** (expected summer 2026 beta).
- **SOGo 5.x** is a single container running the Objective-C `sogod` daemon +
  a database (MySQL/MariaDB/PostgreSQL) + Memcached.
- **SOGo 6** is **three** services (frontend, backend, agent) + PostgreSQL +
  Redis -- see `../evaluation/sogo-6/deployment.md` for the full architecture.
- A reverse proxy (nginx, Traefik, HAProxy) is **required** in both cases for
  TLS termination and CalDAV/CardDAV/ActiveSync routing.
- Never deploy from a Compose file that contains real secrets, pinned digests
  you have not verified, or untested backup/restore procedures.

## When to Use This Guide

| Use case       | Recommendation | Container image                         | Support status                     |
|----------------|----------------|-----------------------------------------|------------------------------------|
| Production     | SOGo 5.x       | `sonroyaalmerol/docker-sogo:5.12.8-2`  | Stable; security patches ongoing   |
| Pilot          | SOGo 6         | Not yet published (placeholder tags)    | Pre-beta; no SLA                   |
| Development    | SOGo 5.x or 6  | Either                                   | Community or subscription-backed   |
| Evaluation     | SOGo 6 only    | Not yet published (placeholder tags)    | Demo instance or self-built        |

> **Production requires a support contract from Alinto** for official binary
> packages. The community Docker image builds from the same GPL source code
> but has no commercial backing. Evaluate your risk tolerance accordingly.

## Reference Stack: SOGo 5.x

The following `docker-compose.yml` provides a self-contained SOGo 5.x stack
with MariaDB and Memcached. Adjust secrets, volume paths, and resource limits
before any production use.

```yaml
# Reference only. Do NOT deploy to production without review, secrets rotation,
# and a tested backup/restore plan.

version: "3.9"

services:
  sogo:
    image: sonroyaalmerol/docker-sogo:5.12.8-2
    container_name: sogo
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
      memcached:
        condition: service_started
    ports:
      - "127.0.0.1:20000:20000"   # sogod internal port; reverse proxy fronted
      - "127.0.0.1:80:80"         # nginx inside the container (optional)
    volumes:
      - sogo-mail-data:/srv       # Mail spool, backups, logs
      - ./sogo-config:/etc/sogo/sogo.conf.d:ro   # YAML configs
    environment:
      - DB_HOST=db
      - DB_PORT=3306
      - DB_NAME=sogo
      - DB_USER=sogo
      - DB_PASSWORD=${SOGO_DB_PASSWORD:?err}
      - MEMCACHED_HOST=memcached
      - MEMCACHED_PORT=11211
      # Additional SOGo environment vars (see the image docs):
      # - SOGO_SOGoIMAPServer=imaps://mail.example.org
      # - SOGO_SOGoSMTPServer=smtp://mail.example.org
      # - SOGO_SOGoMailDomain=example.org
      # - SOGO_SOGoUserSources=...
    healthcheck:
      test: ["CMD", "sogod", "-v"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: mariadb:11
    container_name: sogo-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:?err}
      MYSQL_DATABASE: sogo
      MYSQL_USER: sogo
      MYSQL_PASSWORD: ${SOGO_DB_PASSWORD:?err}
    volumes:
      - sogo-db-data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  memcached:
    image: memcached:1-alpine
    container_name: sogo-memcached
    restart: unless-stopped
    command: ["memcached", "-m", "256"]   # 256 MB; adjust to your scale

volumes:
  sogo-mail-data:
  sogo-db-data:
```

> **PostgreSQL alternative:** replace the `db` service with `postgres:16`,
> change environment vars to `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`,
> and use the `sope49-gdl1-postgresql` connector (already included in the
> community image). SOGo 5.x supports MySQL/MariaDB, PostgreSQL, and Oracle.

### nginx vhost snippet (SOGo 5.x)

Place this in your reverse proxy configuration. It handles the main web UI,
CalDAV/CardDAV, and ActiveSync traffic.

```nginx
server {
    listen 443 ssl http2;
    server_name sogo.example.org;

    ssl_certificate     /etc/ssl/certs/example.org.pem;
    ssl_certificate_key /etc/ssl/private/example.org.key;

    # ActiveSync requires large body buffers
    client_body_buffer_size 1M;
    client_max_body_size    100M;
    proxy_buffering         off;
    proxy_request_buffering off;

    # Main SOGo web UI
    location /SOGo/ {
        proxy_pass http://127.0.0.1:20000/SOGo/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 120s;
    }

    # CalDAV / CardDAV (used by macOS, iOS, Thunderbird)
    location /SOGo/dav/ {
        proxy_pass http://127.0.0.1:20000/SOGo/dav/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 120s;
    }

    # Microsoft ActiveSync
    location /Microsoft-Server-ActiveSync {
        proxy_pass http://127.0.0.1:20000/Microsoft-Server-ActiveSync;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 180s;
    }

    # Optional: serve static assets directly
    location /SOGo/so/ {
        proxy_pass http://127.0.0.1:20000/SOGo/so/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        expires 7d;
    }
}
```

## Reference Stack: SOGo 6

The SOGo 6 stack splits the monolith into three containerised components as
described in `../evaluation/sogo-6/deployment.md`. Image tags below are
**placeholders** -- no official SOGo 6 images have been published as of
June 2026.

```yaml
# Reference only. Do NOT deploy to production without review, secrets rotation,
# and a tested backup/restore plan.
# SOGo 6 image tags are PLACEHOLDERS -- verify availability before use.

version: "3.9"

services:
  frontend:
    image: ghcr.io/sogo/frontend:6.0.0-beta   # PLACEHOLDER
    container_name: sogo6-frontend
    restart: unless-stopped
    depends_on:
      backend:
        condition: service_started
    environment:
      BACKEND_URL: http://backend:5000
    ports:
      - "127.0.0.1:3000:3000"   # Next.js default port
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/SOGo/"]
      interval: 30s
      timeout: 10s
      retries: 3

  backend:
    image: ghcr.io/sogo/backend:6.0.0-beta    # PLACEHOLDER
    container_name: sogo6-backend
    restart: unless-stopped
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    environment:
      DB_URL: postgresql://sogo:${SOGO6_DB_PASSWORD:?err}@db:5432/sogo6
      CACHE_URL: redis://cache:6379/0
      SECRET_KEY: ${SOGO6_SECRET_KEY:?err}
      ENCRYPTION_KEY: ${SOGO6_ENCRYPTION_KEY:?err}
      # Process settings can also be provided via a mounted file:
      # PROCESS_SETTINGS_FILE: /etc/sogo6/process_settings.env
    volumes:
      - sogo6-backend-data:/data
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  agent:
    image: ghcr.io/sogo/agent:6.0.0-beta      # PLACEHOLDER
    container_name: sogo6-agent
    restart: unless-stopped
    depends_on:
      backend:
        condition: service_started
      db:
        condition: service_healthy
      cache:
        condition: service_started
    environment:
      DB_URL: postgresql://sogo:${SOGO6_DB_PASSWORD:?err}@db:5432/sogo6
      CACHE_URL: redis://cache:6379/0
      BACKEND_URL: http://backend:5000
      SECRET_KEY: ${SOGO6_SECRET_KEY:?err}

  db:
    image: postgres:16
    container_name: sogo6-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: sogo6
      POSTGRES_USER: sogo
      POSTGRES_PASSWORD: ${SOGO6_DB_PASSWORD:?err}
    volumes:
      - sogo6-db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U sogo -d sogo6"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    container_name: sogo6-redis
    restart: unless-stopped
    volumes:
      - sogo6-redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  sogo6-backend-data:
  sogo6-db-data:
  sogo6-redis-data:
```

> **These image tags are placeholders.** Verify availability with the SOGo team
> or wait for the public release before using this Compose in any
> non-evaluation environment.

### nginx vhost snippet (SOGo 6)

The path layout for SOGo 6 is designed to be backwards-compatible with the
SOGo 5 proxy configuration. The same nginx location blocks apply.

```nginx
server {
    listen 443 ssl http2;
    server_name sogo6.example.org;

    ssl_certificate     /etc/ssl/certs/example.org.pem;
    ssl_certificate_key /etc/ssl/private/example.org.key;

    client_body_buffer_size 1M;
    client_max_body_size    100M;

    location /SOGo/ {
        proxy_pass http://127.0.0.1:3000/SOGo/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 120s;
    }

    location /SOGo/dav/ {
        proxy_pass http://127.0.0.1:3000/SOGo/dav/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location /Microsoft-Server-ActiveSync {
        proxy_pass http://127.0.0.1:3000/Microsoft-Server-ActiveSync;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 180s;
    }
}
```

## Production Checklist

- **TLS termination** -- terminate TLS at the reverse proxy. Do not expose
  SOGo's internal HTTP port to the network. Use cert-manager (K8s) or
  certbot/lego (VMs) for automatic certificate renewal.
- **Backup strategy** -- back up the database (pg_dump / mysqldump) **and** the
  mail spool volume (`/srv` on SOGo 5, `/data` on SOGo 6) separately. Test
  restores at least quarterly.
- **Log shipping** -- forward container logs to a central aggregator
  (Loki, ELK, Graylog). SOGo logs to stderr by default in the community image.
- **ActiveSync buffer tuning** -- set `client_body_buffer_size 1M` and
  `client_max_body_size 100M` in the reverse proxy. ActiveSync sends large
  multipart requests (calendar sync, attachments). Insufficient buffer sizes
  cause silent timeouts.
- **Rate-limiting** -- apply `limit_req` / `limit_conn` in nginx, or use
  Traefik's rate-limiting middleware, keyed on the client IP. The login path
  `/SOGo/connect` is the most sensitive.
- **Fail2ban** -- configure fail2ban to watch the SOGo log for repeated
  authentication failures. The default jail regex reads sogod's `WARNING`
  messages on failed login.
- **Monitoring** -- monitor sogod process liveness (healthcheck above), IMAP
  backend reachability (can the container resolve and connect to `mail.example.org:993`),
  and disk usage on the mail spool and database volumes.
- **Upgrade pinning** -- pin to a specific image digest
  (`sonroyaalmerol/docker-sogo@sha256:...`). Pull with
  `docker compose pull --quiet` and verify the digest before restarting.
- **Security updates** -- subscribe to the SOGo mailing list or watch the
  [Alinto/sogo](https://github.com/Alinto/sogo) GitHub repository for
  security advisories. Rebuild or re-pull the image when a new patch is
  released.
- **Network isolation** -- create a dedicated `docker network` for backend
  inter-service traffic. Only the reverse proxy should be on the public-facing
  network.
- **Secrets management** -- store secrets in a `.env` file (never committed),
  Docker secrets (`docker secret create`), or an external vault
  (HashiCorp Vault, AWS Secrets Manager). **Never** hardcode credentials in
  the Compose file.

## Troubleshooting

| Symptom                                         | Likely cause                                           | Fix                                                                                                      |
|-------------------------------------------------|---------------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| ActiveSync on Android deletes events            | URL encryption in SOGo                                  | See `../evaluation/sogo-6/known-limitations.md`. Consider temporarily disabling `SOGoURLEncryptionEnabled` |
| Login returns 500 after upgrade                 | Auth source misconfiguration                            | Check sogod log; verify `SOGoUserSources` JSON in the admin UI or config file                            |
| CalDAV clients get 302 redirects                | SSO/nginx auth path not open to DAV routes              | Mirror the DAV location block (`/SOGo/dav/`) in your reverse proxy; ensure it is not behind auth middleware |
| MIME type errors in browser console on SOGo 6   | Known SOGo 6 demo bug (next/font / CSS strict checking) | Track `../evaluation/sogo-6/known-limitations.md`; typically cosmetic, not data-loss                       |
| SOGo won't start: `unable to connect to memcached` | Memcached not reachable or wrong port                   | Confirm `MEMCACHED_HOST`/`MEMCACHED_PORT` in environment; check container networking                     |
| ActiveSync sync fails with HTTP 413             | `client_max_body_size` too small                        | Increase `client_max_body_size` to at least 100M in the reverse proxy                                    |

## Kubernetes

- **Helm chart landscape:** there is no widely-adopted upstream chart at the
  time of writing. The community image
  ([`sonroyaalmerol/docker-sogo`](https://github.com/sonroyaalmerol/docker-sogo))
  ships a beta Helm chart at `https://helm.snry.xyz/docker-sogo/` -- do not
  use it in production without thorough review.
- **StatefulSet for the database** -- both MariaDB and PostgreSQL have mature
  operators (mariadb-operator, CloudNativePG, Zalando Postgres Operator). Use
  one instead of baking the DB into the Compose model.
- **PersistentVolume for mail spool** -- SOGo 5.x stores mail data under
  `/srv`; SOGo 6 uses `/data`. Provision a ReadWriteOnce PVC with sufficient
  IOPS for your user base. NFS-backed storage is **not recommended** for mail
  spools (locking issues).
- **Ingress** -- SOGo needs to serve TCP connections. For ActiveSync, ensure
  the Ingress controller (nginx-ingress, Traefik) is configured with
  `nginx.ingress.kubernetes.io/proxy-body-size: 100m` and
  `nginx.ingress.kubernetes.io/proxy-buffering: "off"` as annotations.
- **Secrets** -- use ExternalSecrets or the native `Secret` resource. The SOGo
  community image supports YAML configuration mounted from a ConfigMap or
  Secret.
- **Liveness / readiness probes** -- SOGo 5.x: probe `GET /SOGo/` on port
  20000. SOGo 6: probe `/health` on the backend and `GET /SOGo/` on the
  frontend.
- **Replicas** -- SOGo 5.x supports multiple replicas behind a shared
  Memcached and database. SOGo 6's stateless frontend and backend can scale
  horizontally; the Celery agent requires careful Redis broker configuration
  for task deduplication.
- **Resource requests** -- start with 1 CPU / 2 GB RAM for the SOGo container
  and adjust based on active user count and concurrent ActiveSync sessions.

## References

- [SOGo 6 Known Limitations](../evaluation/sogo-6/known-limitations.md) --
  active blockers and workarounds
- [SOGo 6 Deployment Architecture](../evaluation/sogo-6/deployment.md) --
  three-service breakdown
- [SOGo 6 Feature Comparison](../evaluation/sogo-6/features.md) -- 5.x vs 6.x
- [Alinto/sogo (GitHub)](https://github.com/Alinto/sogo) -- upstream source
- [SOGo Home](https://sogo.nu) -- project homepage, download, FAQ
- [SOGo Wiki](https://wiki.sogo.nu) -- community-contributed guides
- [SOGo Bug Tracker](https://bugs.sogo.nu) -- report and track issues
- [Alinto Packages](https://packages.sogo.nu) -- official binary packages
- [sonroyaalmerol/docker-sogo](https://github.com/sonroyaalmerol/docker-sogo) --
  community Docker image for SOGo 5.x
- [SOGo May 2026 News -- Deployment](https://www.sogo.nu/news/2026/sogo-v6-may.html) --
  Alinto's own guidance on SOGo 6 containerisation

---

Maintained by the SOGo Community of Practice. Not affiliated with the SOGo project.
