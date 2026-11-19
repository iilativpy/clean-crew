# API and token security

Open when building or reviewing a REST/JSON API, token handling (sessions, JWT, API keys,
OAuth2/OIDC), rate limiting, or a server that calls other servers.

## The control pipeline for each request
Map each control to the threat it answers (STRIDE) and keep this **order**:
1. **Rate limiting / load shedding** (denial of service): the very first decision, so overload is
   rejected cheaply, before authentication or parsing costs resources. Enforce at the proxy or
   gateway **and** again in the app (defense in depth). Return `429` with `Retry-After`. Never let
   unauthenticated requests consume significant resources.
2. **TLS** (information disclosure, tampering): HTTPS only, HSTS for browsers, certificate and
   hostname verification on for outgoing calls. TLS gives no availability protection.
3. **Authentication** (spoofing): establishes who is calling. A failed authentication does not by
   itself reject the request; the later access-control decision does, so unauthenticated and failed
   requests are still audited.
4. **Audit logging** (repudiation): record who did what, when and with what outcome. Log failed
   and denied requests too. Place it after authentication so it can name the caller, and before
   access control so denials are recorded.
5. **Access control** (information disclosure, tampering, elevation of privilege): function-level
   and object-level, decided on the server for every request (see `SKILL.md` section 6).
Use a model such as an access-control list or role set that the code can show and test.

## Status codes and error bodies
- `401` means "not authenticated"; `403` means "authenticated but not allowed"; `404` when even
  the existence of the object must stay hidden; `405` for a method not allowed (use a method
  allowlist); `413` too large; `415` unsupported content type; `429` rate limited.
- Error bodies are generic with an error id. No stack traces, SQL or paths.
- Require a supported `Content-Type` on bodies, never echo `Accept` into `Content-Type`.

## Input and data handling
- Strong typing for every field: type, length, range, format, enum. Reject unknown fields or ignore
  them explicitly. Bind bodies to an allowlisted DTO (no mass assignment).
- Hard size limits on bodies, arrays, string lengths, nesting depth and page size. Cap
  pagination and bound every list endpoint. Allowlist sort and filter fields.
- State-machine rules (order of calls, allowed transitions) are enforced on the server.
- Idempotency keys for payments and other non-repeatable actions.
- Never put credentials, tokens or API keys in URLs; use headers or the body.
- Responses return only the fields the caller may see (DTO per audience), not whole model objects.

## Passwords and API keys
- Passwords: slow salted hash (Argon2id preferred; scrypt; bcrypt legacy). Example minimums:
  Argon2id m=19 MiB, t=2, p=1 (or m=46 MiB, t=1); scrypt N=2^17, r=8, p=1; bcrypt cost 10 or
  more (72-byte input limit). Store the algorithm and parameters with the hash and re-hash at login
  as costs rise. Tune to roughly 100 ms to 1 s per hash on your hardware.
- API keys: long random, shown once, stored **hashed**, scoped to the least privilege, revocable,
  rotated, rate-limited per key. A key alone is not enough for sensitive resources.

## Server-side sessions vs. self-contained tokens
- Server-side sessions (opaque id, state in the store) are simplest: easy to revoke, nothing
  sensitive in the client. Choose them unless you need to verify tokens across services without a
  shared store.
- Self-contained tokens (JWT and similar) are **signed, not secret**: do not put sensitive data
  in them, and plan revocation (short lifetime, denylist keyed on `jti`, token status list).

## JWT rules
- The verifier **hard-codes** the accepted algorithms. Reject `none`. Never read the algorithm
  from the token, and never mix symmetric and asymmetric algorithms in one verifier (key
  confusion).
- Prefer asymmetric signatures (EdDSA, ES256, PS256); RS256 is acceptable only when needed.
  Symmetric (HS256+) needs a random secret at least as long as the hash output, a separate secret
  per issuer/audience pair, and never a password.
- **Never trust key material named in the token** (`jwk`, `jku`, `x5u`, `x5c`). Use pinned keys or
  the issuer's configured key set. `kid` only selects among already-trusted keys, and is
  sanitized; a fetched key URL gets SSRF protection.
- Require and check `iss` (exact match), `aud` (your own identifier), `exp`, and `nbf` if present.
  Set an explicit `typ` and reject others; keep different token types apart by key or audience.
- Short lifetimes. Do not store in `localStorage`; use a `Secure; HttpOnly; SameSite` cookie, or
  keep it in memory with refresh-token rotation.
- JWE: sign then encrypt, allowlist `alg`/`enc`, no compression before encryption, never trust
  claims from an unsigned or unverified token.

## OAuth 2.0 / OpenID Connect
- **Authorization Code + PKCE (S256)** for every client type, including SPAs and native apps. No
  implicit grant, no password grant.
- The server enforces the code verifier whenever a challenge was sent, and rejects a verifier
  with no challenge (downgrade).
- CSRF on the callback: `state` bound to the browser session (or PKCE/`nonce`).
- Redirect URIs matched **exactly**, never by prefix or pattern; no plain `http` except loopback
  for native apps; no open redirectors anywhere on the site.
- Access tokens: short lifetime, audience-restricted (ideally one resource server), least-privilege
  scope. Each resource server checks audience and scope on every call.
- Refresh tokens: rotate with immediate invalidation of the old one, or bind to the client
  (DPoP/mTLS). Prefer asymmetric client authentication (`private_key_jwt`, mTLS) for
  confidential clients.
- OIDC login: validate the ID token (`iss`, `aud`, signature, `exp`, `nonce`); identify the user
  by the pair (`iss`, `sub`), **never by email**. Clients do not choose their own `client_id` or
  `sub`.

## Service-to-service and outgoing requests (SSRF)
- Authenticate services to each other (mTLS, signed tokens with audience); do not rely on being
  on "the internal network".
- When the server fetches a user-supplied URL:
  - allowlist hosts where the set is known, accept only `http`/`https`, build the request
    yourself with fixed scheme, port and path;
  - disable automatic redirects (or re-validate each hop);
  - resolve the name and check **all** returned IPs against a deny set: loopback, link-local
    (including `169.254.169.254` and `fd00:ec2::254`), private ranges, unspecified, multicast,
    IPv4-mapped and NAT64 forms (normalize to IPv4 first); then connect to the checked address
    (defends against DNS rebinding) keeping the original Host/SNI and certificate checks;
  - reject a URL when two parsers would read different hosts (`http://a.com\@evil.com`);
  - put egress firewall rules and cloud metadata protection (IMDSv2) underneath as another layer.
- Webhooks you send: validate the destination the same way, sign the payload, limit the response
  size and time.

## Management and ancillary endpoints
- Management, metrics, debug and health endpoints are not public, or sit behind strong
  authentication on a separate port/host.
- CORS off unless needed; explicit origins when on (see `web-and-browser.md`).
- Security headers for browser-consumed APIs: `Cache-Control: no-store`,
  `X-Content-Type-Options: nosniff`, `Content-Security-Policy: frame-ancestors 'none'`, HSTS.
