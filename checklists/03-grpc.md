# gRPC Security Checklist

gRPC's strong typing and code generation cover some attack surface but introduce others.

## Transport

- TLS required, mutual TLS where appropriate (service-to-service).
- ALPN configured (`h2`).
- Strong cipher suites; disable legacy.

## Authentication

- For external gRPC: token-based auth (JWT in metadata is common). Validate signature, expiry, audience.
- For service-to-service: mTLS with proper certificate validation, including expiry and revocation.
- Is the certificate authority chain pinned where appropriate?

## Authorization

- gRPC interceptors for authz checks, applied uniformly across all RPCs?
- Method-level checks: each `service Foo { rpc Bar }` gets its own authz, not a global "authenticated" check.
- For streaming RPCs: is authz checked on the initial call only, or also as messages arrive (relevant for long-lived streams where the user's permissions might change)?

## Resource limits

- `max_receive_message_length` and `max_send_message_length` set explicitly. The default (4 MB) is often too high for some RPCs and too low for others, set per service.
- `max_concurrent_streams` configured.
- Keep-alive parameters tuned (avoid keep-alive flooding DoS).
- Per-RPC timeouts enforced server-side, not just client-suggested.

## Reflection

- gRPC reflection (`grpc.reflection.v1alpha.ServerReflection`) exposes service definitions. Disable in production for external endpoints.

## Error handling

- Don't leak internal errors via `grpc.Status` details. Map server-side exceptions to safe codes.

## Streaming-specific

- Server streaming: cap output rate or total bytes per stream.
- Client streaming: cap inbound rate / message count.
- Bidi: both of the above, plus a way to close idle streams.

## Code generation

- Are generated stubs reviewed for accidental exposure (e.g., generating stubs from an internal proto into a public package)?
- Are proto changes reviewed for backwards-incompatible field changes that could cause clients to misinterpret data?

## Testing

- Use `grpcurl` to manually probe endpoints. Try unauthenticated calls. Try malformed messages.
- Fuzz with `grpc-fuzz` or similar for input validation issues.
- Load test with realistic message sizes to catch DoS edges.
