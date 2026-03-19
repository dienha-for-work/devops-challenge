## What problems were found

The system exhibited intermittent API failures due to several issues:
- API container is not ready when Nginx starts
- depends_on only ensured startup order, not readiness.
- Nginx attempted to route traffic before the API was fully initialized.
- No health checks across services
- API, PostgreSQL, and Redis had no readiness validation.
- This caused race conditions during startup.
- Database and cache readiness not enforced
- API could start before PostgreSQL and Redis were ready.
- Nginx misconfiguration
- Incorrect upstream port (3001 instead of 3000)
- Missing essential proxy headers:
- Host
- X-Real-IP
- X-Forwarded-For
- X-Forwarded-Proto
- Missing tooling for health checks
- API container lacked curl, preventing HTTP-based health checks.

## How the issues were diagnosed
- Intermittent failures pointed to timing issues rather than complete outages.
- Startup behavior testing
  - Observed that:
    - Containers started in order
    - But services were not actually ready
- Configuration review
  - Found:
    - No health checks defined
    - Incorrect Nginx upstream port
    - Missing proxy headers
- Reproduction
  - Failures occurred during cold starts when:
  - Nginx routed requests too early

## Fixes applied

### Added curl to support container health checks:
```Dockerfile
+ RUN apk add curl
```

### Health checks added to all services

**Nginx**
``` yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:80/status"]
  interval: 2s
  timeout: 3s
  retries: 5
  start_period: 5s
```
**API**
``` yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/status"]
  interval: 2s
  timeout: 3s
  retries: 5
  start_period: 5s
```
**PostgreSQL**
``` yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 10s
```
**Redis**
``` Dockerfile
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 10s
```

### Enforced service readiness with depends_on

Nginx depends on API health
```yaml
services:
  nginx
    ...
    depends_on:
      api:
        condition: service_healthy
```
API depends on database and cache health
```yaml
services:
  api
    ...
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
```

Corrected API upstream port and added required headers:
```diff
- proxy_pass http://api:3001;
+ proxy_pass http://api:3000;
+ proxy_set_header Host $host;
+ proxy_set_header X-Real-IP $remote_addr;
+ proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
+ proxy_set_header X-Forwarded-Proto $scheme;
```

## What monitoring/alerts you would add

- Health check container monitoring
- Endpoint monitoring
  - Periodically check /status endpoints for API and Nginx
- Alert on:
  - Increased 5xx responses
  - Failed health checks
  - Container restarts

## Prevention strategies for production
- Always define health checks
- Every service must expose a /status or equivalent endpoint
- Validate configurations early
  - Ensure:
    - Correct ports
    - Proper proxy headers
    - Add retry logic in API
- Endsure DB/Redis readiness gracefully
- Test cold starts
- Simulate full system startup in staging
- Adopt production-grade orchestration
- Consider Kubernetes with:
- Readiness probes
- Liveness probes
