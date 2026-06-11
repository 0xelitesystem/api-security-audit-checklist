# REST API Security, OWASP API Top 10 Checklist

Aligned with OWASP API Security Top 10 (2023). Each item has audit questions, common findings, and what acceptable looks like.

## API1:2023, Broken Object Level Authorization (BOLA / IDOR)

**The most common API vulnerability.** Endpoint accepts an object ID (user, document, order) and returns it without checking that the caller owns or has rights to the object.

- Does every endpoint that takes an ID re-check authorization on the resource, not just the user's authentication?
- Are object IDs sequential / guessable (1, 2, 3...) or unguessable (UUIDs)? Sequential is fine if authorization is correct, but it's a smell.
- Test: log in as user A, request user B's resource by ID. Reject expected.
- Test: same with nested resources (`/orgs/X/users/Y/documents/Z`). Each level needs authorization.

## API2:2023, Broken Authentication

- Are authentication tokens scoped, time-limited, and revocable?
- Are password reset flows resistant to enumeration (response and timing should be identical for valid/invalid emails)?
- Is rate limiting in place on login, password reset, and MFA endpoints?
- Are JWTs validated correctly (algorithm enforcement, signature verification, expiry, audience, issuer)? `alg: none` confusion still happens.
- Are sessions invalidated on password change / role change / explicit logout?

## API3:2023, Broken Object Property Level Authorization

Mass assignment and excessive data exposure. The endpoint either accepts properties it shouldn't or returns properties it shouldn't.

- Does the endpoint accept user-supplied JSON and bind to model fields without an allowlist? (`User.update(request.json)` is the classic bug, sets `is_admin: true`.)
- Do response objects include fields the caller shouldn't see (other users' emails, internal flags, password hashes, MFA secrets)?
- Are DTOs / serializers used per-endpoint, not "return the whole model"?

## API4:2023, Unrestricted Resource Consumption

- Is there a request rate limit per user / IP / API key?
- Is there a request body size limit?
- For pagination: is there a maximum page size? (Default to 100, cap at 1000, reject larger.)
- For uploads: file size limit, total storage quota per user.
- For expensive operations (search, export, report generation): are they queued, rate-limited separately, or reserved for premium tiers?

## API5:2023, Broken Function Level Authorization

Endpoint exists for admin use but isn't checked. Common in poorly versioned APIs (`/admin/users` is protected but `/v2/users/promote` isn't).

- Are admin/internal endpoints behind a separate authorization check, not just "is authenticated"?
- Are there separate gateways or path prefixes for admin vs user APIs, with consistent enforcement?
- Is there a documented inventory of every endpoint with its required role?

## API6:2023, Unrestricted Access to Sensitive Business Flows

Functionality that's secure per-call but abusable in aggregate: bulk account creation, gift card brute-force, scalping, pricing scraping.

- Have you identified your sensitive business flows? Examples: signup, password reset, purchase, refund, friend-add, message-send.
- Is there protection against automation: CAPTCHAs, device fingerprinting, anomaly detection?
- Are there sensible per-account limits on these actions?

## API7:2023, Server-Side Request Forgery (SSRF)

- Does the API ever fetch URLs supplied by the caller (webhooks, image URLs, "fetch from URL" features)?
- Are private IP ranges (10/8, 172.16/12, 192.168/16, 127/8, 169.254/16, IPv6 equivalents) blocked?
- Is the cloud metadata endpoint (`169.254.169.254`) blocked specifically?
- Are redirects followed safely (re-validate after each hop)?
- DNS rebinding: is the hostname re-resolved at fetch time, or trusted from initial validation?

## API8:2023, Security Misconfiguration

- TLS 1.2+ only, modern cipher suites, HSTS.
- Verbose error messages disabled in production (no stack traces, no SQL errors).
- Unused HTTP methods rejected (DELETE on read-only endpoint should 405, not silently accept).
- CORS configured per-endpoint, not `Access-Control-Allow-Origin: *` with credentials.
- Default credentials removed.

## API9:2023, Improper Inventory Management

- Is there an authoritative API inventory (OpenAPI / similar) that matches what's deployed?
- Are deprecated endpoints removed, or do they live forever serving as forgotten attack surface?
- Are non-production environments (staging, QA) on separate domains and not indexed by search engines?

## API10:2023, Unsafe Consumption of APIs

- For third-party API responses you consume: do you validate the response shape and content?
- Do you trust third-party redirects, cached data, or callbacks without verification?
- Are credentials for third-party APIs scoped narrowly and rotated?
