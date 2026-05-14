# BOLA / IDOR Deep Dive

Broken Object Level Authorization is the single most common API vulnerability and worth a dedicated checklist beyond the OWASP top 10 entry.

## Patterns

- **Direct ID:** `/api/orders/12345` — caller owns order 12345 only if check is present.
- **Embedded in body:** `POST /api/transfer { from: "acct123", to: "acct456" }` — does the caller own `from`?
- **Nested:** `/api/orgs/abc/projects/xyz/issues/123` — each level needs to be checked, and consistency between levels must be verified (does project xyz belong to org abc?).
- **GUID-prefix bypass:** if IDs are partially predictable (e.g., timestamp prefix), authorization based on possession of full ID is weak.
- **Filter parameters:** `GET /api/users?manager_id=me` — does the API verify that listed users belong to the caller's purview, or trust the filter?
- **Mass operations:** `DELETE /api/items?ids=1,2,3,4` — checked per-ID or just the first?
- **Imports/exports:** bulk endpoints often skip per-row checks present in single-row endpoints.

## How it gets missed

- New endpoint added; copy-pasted from another endpoint with different ownership semantics; check left out.
- Authorization in middleware checks that the user is authenticated but not what they can access.
- Test suite covers happy path; cross-account tests are not in CI.
- Multi-tenant systems where the tenant ID is implicit from the user's session — endpoint accepts a tenant ID parameter and uses it instead.

## Audit approach

- Inventory every endpoint that accepts an identifier (path parameter, query parameter, body field).
- For each, identify the resource and the rule for who can access it.
- Write tests that:
  - Authenticate as user A.
  - Attempt to access user B's resource.
  - Expect 403 / 404 (not 200).
- Run these as a separate test suite that runs in CI on every PR.

## Specific test cases to maintain

- Cross-tenant: user from tenant A requests resource from tenant B.
- Same-tenant cross-user: user A in tenant T requests user B's resource (where B is in T).
- Privilege escalation: regular user requests resource gated to admin role.
- Public-vs-private: non-authenticated request for a resource the user thinks is shared but isn't.
- Soft-deleted: request a record that was deleted but still exists in DB.
- ID enumeration: request a range of IDs to see which return 200 vs 404 vs 403. Inconsistent responses leak existence information.

## Mitigation patterns

- Pass user context everywhere; ORM-level row filters tied to user (e.g., `current_user.orders.find(id)` rather than `Order.find(id)`).
- ABAC (attribute-based access control) where complex rules apply.
- Avoid sequential IDs for sensitive resources — UUIDs reduce enumeration but are not authorization.
- Consistent response codes: don't return 404 for "doesn't exist" and 403 for "exists but you can't see it" — unify to one (usually 404) to avoid existence disclosure.

## When you find one

- Assume there are more. BOLA bugs cluster.
- Search the codebase for the same authorization pattern used elsewhere.
- Add the failing test to the regression suite before fixing.
- Check logs for past exploitation. (Often hard to detect; pattern is "user A's session repeatedly accessing user B's resource IDs".)
