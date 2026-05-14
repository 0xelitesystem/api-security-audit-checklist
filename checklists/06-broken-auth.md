# Authentication Checklist

Specific authentication failures that bypass even otherwise-correct authorization.

## Token validation

### JWT

- Algorithm enforced server-side. Don't accept `alg: none`. Don't accept `alg` from the token itself — pin it in code.
- Signature actually verified, not just decoded.
- Standard claims checked: `exp`, `nbf`, `iss`, `aud`.
- Public keys retrieved from a trusted source (JWKS endpoint with TLS) and cached with reasonable TTL.
- Key rotation handled: when keys rotate, do tokens signed with the old key get rejected immediately, or after a grace period?
- Refresh token rotation: each use of a refresh token issues a new refresh and invalidates the previous. Reuse of an old refresh token is a sign of theft and should invalidate the chain.

### Opaque tokens

- Stored hashed in DB, not plaintext.
- Lookup is constant-time where possible.
- Revocation works (test it).

## Login flow

- Rate limit per username AND per IP. Per-IP alone is bypassed by botnets; per-username alone is bypassed by spraying.
- Generic error messages: "invalid email or password" not "user not found" / "wrong password".
- Timing-equivalent responses for valid vs invalid usernames (use a constant-time comparison; perform the password hash even when the user doesn't exist, to equalize timing).
- Track failed login counts; lock after threshold (with sane unlock — usually time-based, not "must contact support").

## Password reset

- Reset tokens are single-use, time-limited (1 hour is generous), unguessable.
- Reset endpoint reveals nothing about whether the email exists. "If an account exists with that email, a reset link has been sent."
- After reset, all existing sessions for that user invalidated.
- Reset link sent over TLS-only channels (HTTPS link, email body that warns against forwarding).

## MFA

- TOTP / WebAuthn / push-based, not SMS where possible.
- Backup codes available, single-use, hashed in storage.
- "Remember this device" tokens scoped to that device, time-limited, revocable.
- MFA bypass paths inventoried: account recovery, reset, support process. Each is a potential bypass.

## Session management

- Sessions invalidated on: password change, MFA setting change, role change, explicit logout.
- Session ID rotates on privilege change (login, sudo).
- Idle timeout and absolute timeout both enforced.
- Cookies: `Secure`, `HttpOnly`, `SameSite=Lax` or `Strict`, `__Host-` prefix where applicable.

## OAuth / SSO

- State parameter required and validated to prevent CSRF.
- Authorization code flow with PKCE for public clients.
- Redirect URI validated against an exact-match allowlist (not regex with wildcards).
- Token endpoint requires client authentication for confidential clients.
- ID token signature validated; nonce checked.

## Common bypasses to test

- JWT with `alg: none` accepted? (`{"alg":"none","typ":"JWT"}.{...}.`)
- JWT with HS256 signed using the public key as HMAC secret? (Some libraries fail open here.)
- Reset token reuse: try the same token twice.
- Reset token from one user applied to another (if predictable).
- MFA bypass via password reset (does reset complete a session without MFA challenge?).
- Race condition on signup / password reset to claim someone else's email.
- OAuth `redirect_uri` accepting subdomain wildcards or path traversal.
