# GraphQL Security Checklist

GraphQL has specific concerns that REST checklists don't fully cover.

## Query depth and complexity

- Is there a maximum query depth limit? (Common: 10-15. Anything deeper is suspicious.)
- Is there a query complexity scoring system (e.g., `graphql-cost-analysis`, `graphql-query-complexity`)? Reject queries above threshold.
- Test: nested fragments, recursive types, alias-multiplication attacks.

## Introspection

- Is introspection (`__schema`, `__type`) disabled in production for public endpoints?
- For internal/partner endpoints, is introspection authenticated?
- Note: disabling introspection is obscurity, not security — assume attackers know your schema.

## Authorization

- Is authorization enforced at the field/resolver level, not just the entry point?
- Many GraphQL bugs come from "the query was authorized to call `user`, but it descended into `user.adminNotes` which wasn't checked separately".
- Use resolver-level guards or schema directives like `@auth(requires: ROLE_ADMIN)`.

## Batching attacks

- Multiple operations in a single request — is each rate-limited individually?
- "Batched login" is a common bypass: 100 login attempts in one POST evades naive per-request rate limits.
- Limit operations per request and arrays per query.

## Error handling

- Default GraphQL errors leak schema info (typos suggest field names, type errors reveal type structures).
- In production, return generic error messages externally; log details internally.

## Mutations

- Mutations are state-changing. Apply CSRF protection if you accept GraphQL via cookies.
- Each mutation needs its own authorization, not just "the query was authenticated".
- Idempotency keys for mutations that should not be replayed (payments, sends).

## Subscriptions

- WebSocket-based subscriptions need auth at connection time and per-channel.
- Are connections rate-limited and capped per user?
- Is there a maximum number of active subscriptions per connection?

## File uploads

- GraphQL multipart uploads are easy to misconfigure. Apply standard upload protections (size, type, virus scanning).
- The upload endpoint is sometimes a separate code path with weaker checks than the main API.

## Tests

- Send a deeply nested query — is it rejected?
- Send 100 batched login mutations — does rate limiting catch it?
- Query a field as an unauthenticated user that should require auth — is it blocked?
- Query a field as a low-privilege user that should require admin — is it blocked?
- Trigger an error and inspect the response — does it leak schema details?
