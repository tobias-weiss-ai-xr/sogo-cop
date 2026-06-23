# SOGo 6 — Performance Evaluation

> ⚠️ SOGo 6 is pre-beta as of June 2026. Performance benchmarks are not yet available from the SOGo team.

## Areas to Benchmark

### Backend (Python / Flask)

- Request throughput (RPS) under concurrent users
- Memory usage per worker (gunicorn)
- Response latency for API endpoints (mail, calendar, contacts)
- Database query performance (PostgreSQL)
- Redis cache hit rates

### Frontend (Next.js / React)

- Time to First Byte (TTFB)
- First Contentful Paint (FCP)
- Largest Contentful Paint (LCP)
- Time to Interactive (TTI)
- JavaScript bundle size
- Client-side memory usage (long-running sessions)
- Server-side rendering overhead

### Agent (Celery)

- Task queue throughput
- Redis memory usage for queue
- Scheduled task latency

## Comparison Points

- SOGo 5 baseline (same hardware)
- Equivalent groupware solutions (Nextcloud, Kopano, Zimbra)

## Known Changes Affecting Performance

✅ **Positive:** Python backend should be faster for API responses than Objective-C CGI  
⚠️ **Neutral:** React bundle size vs AngularJS — depends on optimisation  
❌ **Risk:** Three containers instead of one — increased resource overhead  
⚠️ **Risk:** Redis dependency adds operational complexity but improves caching  
✅ **Positive:** Celery enables async tasks (scheduled sends) without blocking requests
