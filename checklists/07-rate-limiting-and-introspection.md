# Rate Limiting, Schema Disclosure, and Hardening

## Rate limiting

### What to limit

- Per-user / per-token, not just per-IP. Authenticated abuse should still be limited.
- Per-IP, especially on unauthenticated endpoints.
- Per-API-key for B2B integrations.
- Per-endpoint differently — login deserves tighter limits than `/health`.
- Specifically high-cost endpoints: search, export, report, image processing, anything that calls a third-party service.

### How to fail

- 429 status code with `Retry-After` header.
- Headers exposing current limit / remaining / reset time (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`).
- Don't leak limit values that help attackers tune their rate.

### Specific tests

- Burst tests (1000 requests in 1 second) — does the limit kick in?
- Distributed tests (100 IPs, 10 each) — does per-user / per-token limit catch what per-IP doesn't?
- Resource-specific limits: try to brute-force a specific user's password from across IPs. Does per-username limiting catch it?

## Schema and information disclosure

### Public-facing OpenAPI

- Is your OpenAPI spec public? Decide deliberately. If yes, scrub internal-only fields, deprecated endpoints, debug routes.
- Auto-generated specs from code can include endpoints you forgot existed — review before publishing.

### Error messages

- No stack traces, SQL errors, or internal hostnames in production responses.
- Validation errors describe the issue without revealing schema details ("invalid input" rather than "expected `internal_admin_flag` to be boolean").
- 404 and 403 responses are consistent — don't reveal resource existence through differential responses (see also BOLA checklist).

### Response headers

- Strip `Server`, `X-Powered-By`, framework version headers.
- `Cache-Control` set appropriately (sensitive responses should be `no-store`).
- Add `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`.
- For JSON APIs that won't be called from browsers, restrictive `Content-Security-Policy` and `X-Frame-Options: DENY`.

## CORS

- Specific origins listed, not `*` with credentials.
- `Access-Control-Allow-Credentials: true` requires very careful origin allowlisting; one mistake means cross-origin theft of authenticated responses.
- Preflight (`OPTIONS`) responses cached appropriately.
- Don't reflect the `Origin` header back without validation.

## Logging and monitoring

- Authentication failures logged with: timestamp, source IP, user identifier (if known), endpoint.
- Authorization failures logged.
- Unusual patterns (high 401 rate, geographic anomalies) trigger alerts.
- Logs scrubbed of sensitive data: no passwords, no tokens, no full payment card numbers, no social security numbers.
- Retention long enough for incident investigation (90+ days typical) but not indefinite.

## API key management

- Keys are unguessable (sufficient entropy).
- Stored hashed in the database.
- Display the full key once on creation, then only a prefix. (`sk_live_abc...xyz` style.)
- Per-key scopes / permissions, rotation support, revocation.
- Audit log of key usage available to the customer.
- Compromised-key detection: regular scans of GitHub, Pastebin, etc. for your key prefixes.

## Headers worth setting on every API response

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
Cache-Control: no-store
Content-Security-Policy: default-src 'none'
X-Frame-Options: DENY
Referrer-Policy: no-referrer
```
