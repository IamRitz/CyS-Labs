# Lab 1 — Submission

## Triage Report: OWASP Juice Shop

### Scope & Asset
- Asset: OWASP Juice Shop (local lab instance)
- Image: `juice-shop:latest` (v20.2.0: locally built from cloned repository)
- Image digest: sha256:a2f0f1b834356117a92a1f56c9d54a8acc52718004e6a088523e5174ffa3d6f6
- Host OS: Arch-Linux 
- Kernel: 7.1.11-arch1-1
- Docker version: Docker version 29.7.2, build a7dcaa6fdb

### Deployment Details
- Local Build Command: `docker build -t juice-shop .` (from cloned repo)
- Run command used: `docker run -d --name juice-shop -p 127.0.0.1:3000:3000 juice-shop` 
- Access URL: http://127.0.0.1:3000
- Network exposure: 127.0.0.1 only? [X] Yes [ ] No (explain if No)
- Container restart policy: no 

### Health Check
- HTTP code on `/`: 200 
- API check (first 200 chars of `/api/Products`):
  ```
    {"status":"success","data":[{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-09-01T09:01:36.235Z"
  ```
- Container uptime: 7377da540ef1   juice-shop   "/nodejs/bin/node /j…"   22 hours ago   Up 17 minutes   127.0.0.1:3000->3000/tcp   juice-shop

### Initial Surface Snapshot (from browser exploration)
- Login/Registration visible: [X] Yes [ ] No — notes: Login and user registration functionality are exposed through the application UI. 
- Product listing/search present: [X] Yes [ ] No — notes: The application exposes a product catalogue with search functionality.
- Admin or account area discoverable: [ ] Yes [X] No — No obvious admin or account functionality was discoverable during initial unauthenticated browser exploration.
- Client-side errors in DevTools console: [ ] Yes [X] No — No obvious client-side errors were observed during initial browser exploration.
- Pre-populated local storage / cookies: 
  - Local Storage: no application-specific values observed.
  - Session Storage: `__darkreader__wasEnabledForHost=false` observed; appears browser-extension related.
  - Cookies: `language=en`, `welcomebanner_status=dismiss`.

### Security Headers (Quick Look)
Run: `curl -I http://127.0.0.1:3000 2>&1 | head -20`. Paste output:
    ```
    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: *
    X-Content-Type-Options: nosniff
    X-Frame-Options: SAMEORIGIN
    Feature-Policy: payment 'self'
    X-Recruiting: /#/jobs
    Accept-Ranges: bytes
    Cache-Control: public, max-age=0
    Last-Modified: Tue, 01 Sep 2026 09:01:36 GMT
    ETag: W/"2588-1a05c33d7bd"
    Content-Type: text/html; charset=UTF-8
    Content-Length: 9608
    Vary: Accept-Encoding
    Date: Tue, 01 Sep 2026 09:38:23 GMT
    Connection: keep-alive
    Keep-Alive: timeout=5
    ```
Which of these are MISSING? (cross-reference Lecture 1 OWASP Top 10:2025 — A06)
- [X] `Content-Security-Policy`
- [X] `Strict-Transport-Security`
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options`

### Top 3 Risks Observed (2-3 sentences each, in your own words)

1. **Incomplete security-header hardening** — The application does not return
   `Content-Security-Policy` or `Strict-Transport-Security`, while
   `X-Content-Type-Options: nosniff` and `X-Frame-Options: SAMEORIGIN` are
   present. Missing security headers reduce browser-side defense-in-depth
   and should be reviewed for production deployment. **OWASP:
   A02:2025 — Security Misconfiguration.**

2. **Permissive cross-origin policy** — The application/API returns
   `Access-Control-Allow-Origin: *`, allowing cross-origin browser access.
   The observed `/api/Products` endpoint currently exposes product catalogue
   information, so the security impact of this configuration is not yet
   established and authenticated/sensitive endpoints should be tested.
   **OWASP: A02:2025 — Security Misconfiguration.**

3. **HTML-bearing product data / potential injection surface** — Product
   descriptions returned by `/api/Products` contain HTML markup such as
   `<a>` and `<em>` elements. This makes output encoding and sanitization
   worth testing for injection/XSS, but no exploitable XSS has been
   established from this initial observation. **Potential OWASP mapping:
   A05:2025 — Injection.**

## PR Template Setup

- File: `.github/PULL_REQUEST_TEMPLATE.md`
- Sections included: Goal / Changes / Testing / Artifacts & Screenshots
- Checklist items: 
 - Title is clear (feat(labN): <topic> style)
 - No secrets/large temp files committed
 - Submission file at submissions/labN.md exists
- Auto-fill verified: [X] Yes — PR description showed my template (screenshot or link to draft PR)

## Bonus: CI Smoke Test

- Workflow file: `.github/workflows/lab1-smoke.yml`
- Trigger: `pull_request` on main
- Run URL: https://github.com/IamRitz/CyS-Labs/actions/runs/33596650171
- Workflow run duration: 19s

- Curl response excerpt:
    GET / HTTP/1.1
    Host: localhost:3000
    User-Agent: curl/8.5.0
    Accept: /
    <
    < HTTP/1.1 200 OK
    < Content-Type: text/html; charset=UTF-8
    < Content-Length: 9393    GET / HTTP/1.1
