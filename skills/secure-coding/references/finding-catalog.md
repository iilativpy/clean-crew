# Security finding catalog

Use these IDs when reviewing or auditing, as the smells catalog is used for code quality. The
"Default" column is the usual severity when the flaw is reachable by an attacker; adjust it for
exploitability and impact (an unauthenticated flaw on money or personal data is higher, a flaw
behind an admin login on a throwaway tool is lower). CWE numbers are for cross-reference with
scanners and reports.

## SEC-INJ - injection and unsafe interpreters
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-INJ-1 | SQL built by string concatenation/formatting with any outside value (also second-order) | 89 | Critical |
| SEC-INJ-2 | NoSQL/LDAP/XPath query or filter built from raw request data (operators injectable) | 943 / 90 / 643 | High |
| SEC-INJ-3 | OS command built from outside data, or a shell used where an argument list would do | 78 | Critical |
| SEC-INJ-4 | `eval`/`exec`/dynamic include/runtime-compiled template with outside data | 94 | Critical |
| SEC-INJ-5 | Path built from outside data without containment check (traversal, absolute path, symlink) | 22 | High |
| SEC-INJ-6 | XML parsed with external entities or DTDs enabled | 611 | High |
| SEC-INJ-7 | Untrusted data deserialized with a type-instantiating format | 502 | Critical |
| SEC-INJ-8 | Header, log or email-header value contains unchecked CR/LF | 113 / 117 | Medium |
| SEC-INJ-9 | Regex with nested quantifiers on unbounded user input | 1333 | Medium |
| SEC-INJ-10 | Template injection: user text compiled as a template | 1336 | Critical |

## SEC-OUT - output and browser
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-OUT-1 | User data in HTML/JS/attribute/URL output without context encoding (reflected, stored, DOM) | 79 | High |
| SEC-OUT-2 | Escape hatch used on untrusted data (`|safe`, `innerHTML`, `v-html`, `dangerouslySetInnerHTML`) | 79 | High |
| SEC-OUT-3 | Home-made HTML filter or blocklist instead of a maintained sanitizer/allowlist | 79 / 184 | High |
| SEC-OUT-4 | User-controlled URL in `href`/`src`/redirect without scheme/host allowlist (`javascript:`) | 79 / 601 | Medium |
| SEC-OUT-5 | Open redirect to a user-supplied target | 601 | Medium |
| SEC-OUT-6 | Sensitive data echoed back (prefilled password, full card number, secret in response) | 200 | Medium |

## SEC-WEB - cross-site and browser controls
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-WEB-1 | State-changing request without CSRF protection (cookie auth, no token/Origin check) | 352 | High |
| SEC-WEB-2 | State change on `GET` | 352 | Medium |
| SEC-WEB-3 | Session/auth cookie missing `Secure`, `HttpOnly` or `SameSite`, or scoped too widely | 614 / 1004 / 1275 | Medium |
| SEC-WEB-4 | CORS reflects arbitrary origin, or `*` with credentials | 942 | High |
| SEC-WEB-5 | No framing protection on sensitive pages (clickjacking) | 1021 | Low |
| SEC-WEB-6 | Missing/weak CSP, `nosniff`, HSTS, `Referrer-Policy` on pages with sensitive data | 693 | Low |
| SEC-WEB-7 | Secrets or tokens in URLs (leak via logs, Referer, history) | 598 | Medium |

## SEC-AUTH - authentication
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-AUTH-1 | Passwords stored in plaintext, reversibly, or with a fast/unsalted hash | 256 / 916 / 759 | Critical |
| SEC-AUTH-2 | No throttling on login, reset, change-password or other credential checks | 307 | High |
| SEC-AUTH-3 | Account enumeration: different message, status or timing for known/unknown account (login, register, reset) | 203 / 204 | Medium |
| SEC-AUTH-4 | Hard-coded, default or shared credentials/keys | 798 | Critical |
| SEC-AUTH-5 | Authentication check that fails open (exception or missing branch lets request through) | 287 / 636 | Critical |
| SEC-AUTH-6 | Critical function reachable without authentication | 306 | Critical |
| SEC-AUTH-7 | Weak reset/recovery: guessable, reusable, non-expiring token; Host-header link; auto-login; secret questions | 640 | High |
| SEC-AUTH-8 | Password change without current password, or accepts a username parameter | 620 | High |
| SEC-AUTH-9 | Weak password policy (very short max length, truncation, no breached-password check) | 521 | Low |
| SEC-AUTH-10 | Credentials sent in URL, cookie, over plain HTTP, or logged | 319 / 532 | High |
| SEC-AUTH-11 | Multistep login whose progress is client-controlled or skippable | 288 | High |
| SEC-AUTH-12 | Identity key not unique: duplicate usernames or emails allowed (accounts blur, resets and sessions hit several rows) | 694 / 287 | Medium |
| SEC-AUTH-13 | Reset token or one-time code reusable, not invalidated after use, or never expires | 640 / 613 | High |
| SEC-AUTH-14 | Sessions not invalidated after password change or reset | 613 | Medium |

## SEC-SES - sessions and tokens
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-SES-1 | Predictable session/reset/API token (weak PRNG, sequence, timestamp, meaningful fields) | 330 / 338 | High |
| SEC-SES-2 | No new session id after login or privilege change (fixation) | 384 | Medium |
| SEC-SES-3 | No server-side logout/expiry (idle or absolute); token valid after logout | 613 | Medium |
| SEC-SES-4 | Session id in URL, log, or `localStorage`; accepted over HTTP | 598 / 922 | Medium |
| SEC-SES-5 | JWT: signature unchecked, `alg:none` accepted, algorithm taken from token, key taken from token header | 347 / 327 | Critical |
| SEC-SES-6 | JWT/OAuth token not checked for issuer, audience or expiry; long-lived with no revocation | 345 / 613 | High |
| SEC-SES-7 | OAuth: implicit or password grant, no PKCE, no `state`, loose redirect URI matching | 601 / 352 | High |
| SEC-SES-8 | Sensitive action without re-authentication or confirmation | 306 | Low |

## SEC-ACC - access control
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-ACC-1 | Missing object-level check: user-controlled id reads or changes someone else's object (IDOR/BOLA) | 639 / 862 | High |
| SEC-ACC-2 | Missing function-level check: privileged operation callable by any/lower-role user | 862 / 863 | High |
| SEC-ACC-3 | Mass assignment: request bound to model; client can set role/owner/price/balance | 915 | High |
| SEC-ACC-4 | Access rights taken from a client parameter, header or cookie | 602 / 807 | Critical |
| SEC-ACC-5 | Authorization scattered/per-route with no deny-by-default; new route unprotected | 862 | High |
| SEC-ACC-6 | Privilege escalation via delegation (grant more than you hold, create stronger user) | 269 | High |
| SEC-ACC-7 | Protected static/uploaded files reachable directly by URL | 425 / 552 | Medium |
| SEC-ACC-8 | Multistep process lets later steps run without earlier checks | 841 | Medium |
| SEC-ACC-9 | App uses over-privileged DB/OS/cloud identity (DBA, root, `*` policy) | 250 | Medium |
| SEC-ACC-10 | Search/export/list leaks objects the caller may not open | 200 / 639 | High |

## SEC-CRY - secrets and cryptography
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-CRY-1 | Secret/API key/private key/token committed to source or shipped to the client | 798 / 540 | Critical |
| SEC-CRY-2 | Weak or broken algorithm/mode (MD5/SHA-1 for security, ECB, DES, static IV, reused nonce) | 327 | High |
| SEC-CRY-3 | Non-cryptographic RNG for a security value | 338 | High |
| SEC-CRY-4 | TLS verification disabled, or plain HTTP for sensitive traffic | 295 / 319 | High |
| SEC-CRY-5 | Home-made crypto/token scheme where a library exists | 327 | Medium |
| SEC-CRY-6 | Secret compared with `==` (timing leak) | 208 | Low |
| SEC-CRY-7 | Trusted value (price, role) sent to the client unsigned, or signed without context (replayable) | 602 / 294 | High |

## SEC-ERR - errors, logging, disclosure
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-ERR-1 | Stack trace, SQL, path or debug detail returned to the client; debug mode on | 209 / 489 | Medium |
| SEC-ERR-2 | Secrets, tokens, passwords or full card numbers written to logs | 532 | High |
| SEC-ERR-3 | Security events not logged (auth failures, denied access, sensitive actions) | 778 | Low |
| SEC-ERR-4 | Exception swallowed on a security path (check silently skipped) | 390 / 636 | High |
| SEC-ERR-5 | User input written to logs unsanitized (log forging) | 117 | Low |
| SEC-ERR-6 | Version banners, internal hostnames, comments with sensitive info exposed | 200 | Low |
| SEC-ERR-7 | Unhandled exception on bad input reachable via the public interface (traceback, 500, crash) | 20 / 248 | Medium |

## SEC-LOG - business logic and state
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-LOG-1 | Price/discount/limit taken from the client, or negative/zero/overflow amount accepted | 602 / 20 | High |
| SEC-LOG-2 | Check-then-act race (double spend, double redeem, limit bypass) | 362 / 367 | High |
| SEC-LOG-3 | Side effect performed before all validation passed (code consumed, mail sent, stock taken) | 841 | Medium |
| SEC-LOG-4 | Floating-point money or unchecked integer arithmetic | 681 / 190 | Medium |
| SEC-LOG-5 | Cumulative limit not enforced (refunds beyond paid, transfers beyond balance) | 840 | High |
| SEC-LOG-6 | Replay of a single-use action possible (no idempotency/nonce) | 294 | Medium |
| SEC-LOG-7 | Audit trail can be edited or deleted by users it records | 779 | Medium |
| SEC-LOG-8 | Incomplete cleanup: deleting a record leaves its files, shares or sessions behind | 459 | Low |
| SEC-LOG-9 | Shared mutable state or one database connection used across threads without synchronization (corruption, crash, interleaved transactions) | 362 / 820 | High |

## SEC-FIL - files and outgoing requests
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-FIL-1 | Unrestricted upload: type not allowlisted, executable content served, stored in web root | 434 | High |
| SEC-FIL-2 | Client-supplied filename used for the stored path | 22 / 73 | High |
| SEC-FIL-3 | No size/count/rate limit on upload or decompression | 770 / 409 | Medium |
| SEC-FIL-4 | Server fetches a user-supplied URL without host/IP validation (SSRF), follows redirects | 918 | High |
| SEC-FIL-5 | Upload served inline with a guessable `Content-Type` (script/HTML executes in the origin) | 79 / 434 | High |

## SEC-CFG - configuration, dependencies, availability
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-CFG-1 | Known-vulnerable dependency in use | 1395 | by CVE |
| SEC-CFG-2 | Insecure default: debug on, default account, open admin or management endpoint | 1188 / 489 | High |
| SEC-CFG-3 | No rate or size limit on a costly or unauthenticated endpoint | 770 | Medium |
| SEC-CFG-4 | Unbounded query/pagination/response size | 770 | Medium |
| SEC-CFG-5 | Overly permissive file or object permissions | 732 | Medium |
| SEC-CFG-6 | Unpinned or unverified build/dependency source | 829 | Low |
| SEC-CFG-7 | Limiter or lock-out an outsider can trip for others (anyone can lock a known account, or exhaust a global counter to disable login, registration or reset for everyone) | 645 / 770 | Medium |
| SEC-CFG-8 | No quota or pruning: storage, rows, sessions or audit entries grow without bound per user or per unauthenticated request | 770 / 400 | Medium |
| SEC-CFG-9 | Throttle that checks and records in separate steps, so parallel requests all pass the check (race) | 362 / 307 | Medium |

## SEC-TST - tests
| ID | Finding | CWE | Default |
|---|---|---|---|
| SEC-TST-1 | New security rule has no test that sends the hostile input | - | Major (in a review) |
| SEC-TST-2 | Security test passes with the control removed (cannot fail) | - | Major |
| SEC-TST-3 | Test asserts insecure behavior (expects a traceback, accepts a forged token) | - | Major |

## Severity guide
- **Critical:** remote, unauthenticated or low-privilege, leads to code execution, full data
  access, account takeover, or money loss.
- **High:** attacker with an ordinary account can read or change others' data or escalate
  privilege; or an unauthenticated attacker obtains sensitive data.
- **Medium:** needs a precondition (victim interaction, specific state, extra flaw) or has limited
  impact.
- **Low:** hardening, defense in depth, little direct impact.
- **Info:** observation, not a vulnerability.
Availability-only problems (lock-out, resource exhaustion, kill switches) are normally **Medium**;
raise to High when one request can take the whole service down, or when the target is an admin
or a recovery path.
Raise one level when the flaw is reachable without authentication, or when it chains with
another finding into full account takeover or data loss (a session flaw that lets anyone act as
any user is Critical, not Medium). Lower one level when it needs an admin account or an unlikely
precondition. Always say why you moved it.
State **confidence** separately: *confirmed* (reproduced or the path is fully traced),
*likely* (path traced, one hop assumed), *possible* (pattern present, reachability unproven).
