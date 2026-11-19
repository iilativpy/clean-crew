---
name: secure-coding
description: Code-level security rules for web apps and APIs - input handling, injection, output encoding, authentication, sessions and tokens, access control, secrets and crypto, error handling, logging, business-logic abuse, uploads, dependencies. Use whenever writing, changing or reviewing code that handles user input, credentials, sessions, permissions, files, money, HTTP, SQL, shell commands, crypto or secrets.
---

# Secure Coding - working rules

Apply these whenever code touches a trust boundary. They are defaults: when the project has an
established, safe convention, follow it. A project convention never excuses a real vulnerability.

**Which sections to read.** This file is short enough to read in full (about 280 lines), and
that is the cheapest way to apply it when the change touches several areas. To read less: sections
0-1 always, then only the sections for the areas you touch (2 sinks, 3 output, 4-5 login and
sessions, 6 permissions, 7 secrets, 8 errors and logs, 9 money and state, 10 files, 11 config),
section 12 for the feature you build, and section 13 for the tests. Reviewers also use the
finding IDs (`SEC-...`) and CWE numbers in `references/finding-catalog.md`.

## 0. Ground rules [write, review]
- **The client controls everything it sends**: every parameter, header, cookie, hidden field,
  token, file name and request order. Nothing that arrives is trusted until the server checks it.
  Client-side checks improve usability; they are never a control.
- **The server decides** identity, price, role and ownership, from server-side state (the session,
  the database), never from a field the client can edit.
- **Deny by default, fail closed.** If a check errors out or cannot decide, the answer is no.
- **Defense in depth.** Assume one layer fails (a validator, a firewall, a framework default) and
  make sure a second layer still stops the attack. Least privilege at every layer limits the damage.
- **Use the platform's safe API** instead of building protection by hand: parameterized queries,
  an argument-array process API, auto-escaping templates, the framework's session and CSRF
  support, a maintained password-hashing library. Do not write your own crypto, session tokens,
  HTML sanitizer or parser.
- **Look at every route into a sink**, not only the obvious one: data stored earlier by any user
  (second-order), values from headers, from another service, from a file.

## 1. Input at every boundary [write, review]
- Validate **on the server**, at the point where data first arrives, **and again** where it
  enters a sensitive component (query, shell, template, URL fetch, file path). Each component
  defends itself; a single filter at the edge cannot know every downstream use.
- Prefer, in this order: (1) a design that never passes the value to the dangerous API; (2) an
  **allowlist** of exact values, or a tight pattern (type, length, range, character set);
  (3) context-correct escaping or parameter binding; (4) sanitizing. **Never rely on a blocklist**
  of "bad" strings: it is bypassed by case, encoding, comments, extra characters and NUL bytes.
- **Reject, do not repair.** Stripping `../` or `<script>` can be defeated by input that
  reassembles itself (`....//`, `<scr<script>ipt>`). If the input is invalid, refuse it.
- **Canonicalize before you check, then do not decode again.** Decode (URL, Unicode, HTML) first,
  validate the final form, and use that same value. Double-decoding after a check reopens the hole.
- Check **type, length, range and format**; reject negatives, zero, `NaN`, huge numbers and
  non-ASCII digits where they make no sense. Limit request and field sizes.
- **Semantic checks are separate from syntax.** A well-formed account number still has to belong
  to the caller (see section 6).
- A parse or library error (overflow, bad encoding, malformed row) becomes a domain error with
  context. One bad item must not crash the request or expose a traceback.

## 2. Safe handling per sink [write, review]
- **SQL / NoSQL / LDAP / XPath:** use parameterized queries or the ORM's bound parameters for
  **every** value, including ones that "come from our own database". Stored procedures do not
  help if they build SQL from strings. Table names, column names and `ASC`/`DESC` cannot be bound:
  pick them from a fixed allowlist. Never build a query or a MongoDB filter from raw JSON the
  client sent (operators like `$ne`, `$where` are injectable). The app's DB account gets only the
  rights it needs (no DBA, read-only where possible).
- **OS commands:** do not call a shell with user data. Use the process API that takes an argument
  array and no shell (`subprocess.run([...])` with `shell=False`, `execFile`, `ProcessBuilder`),
  validate the arguments against an allowlist, and prevent a value starting with `-` from being
  read as an option. Prefer a library function over spawning a process.
- **Dynamic code:** never pass user data to `eval`, `exec`, `Function`, template strings compiled
  at runtime, or include/require by name.
- **File paths:** do not build a path from user text if you can avoid it: serve by generated id
  or from a fixed list. Otherwise resolve the final path (`realpath`, canonical path) and check it
  is **inside** the intended base directory; reject `..`, absolute paths, NUL bytes and symlink
  escapes. Run with least filesystem rights.
- **XML:** disable external entities and DTDs in every parser; do not fetch remote resources.
- **Deserialization:** never deserialize untrusted bytes with a format that can instantiate
  arbitrary types (`pickle`, Java native serialization, `BinaryFormatter`, unsafe YAML loaders,
  JSON with polymorphic type names). Use plain data formats (JSON into a fixed DTO) and a safe loader.
- **Outgoing requests (SSRF):** if the server fetches a user-supplied URL, use an allowlist of
  hosts, allow only `http`/`https`, do not follow redirects blindly, resolve the name and check the
  IP is public (not loopback, link-local, private, cloud metadata `169.254.169.254`), and connect
  to the checked address. See `references/api-and-tokens.md`.
- **Redirects:** never redirect to a user-supplied URL. Redirect to a fixed target or an index into
  a list; if a path is accepted, require a same-site relative path and reject anything with a
  scheme or `//`.
- **HTTP headers and logs:** never put user input in a response header or log line unchecked;
  reject CR/LF (and any byte below 0x20).
- **Regular expressions:** avoid patterns with nested quantifiers on user input (catastrophic
  backtracking); cap the input length first.
- **Email and other protocol messages built from user text:** reject newlines in header fields.

## 3. Output and the browser [write, review]
- **Encode for the context at the moment of output**: HTML body, quoted attribute, JavaScript
  string, URL parameter and CSS each need different encoding. Use the template engine's
  auto-escaping; watch its escape hatches (`|safe`, `innerHTML`, `dangerouslySetInnerHTML`,
  `v-html`, `bypassSecurityTrust*`, `raw`).
- Do not insert user data into `<script>`, event-handler attributes, `style`, or URL attributes
  (`href`, `src`) without an allowlist of schemes (`http`, `https`). Prefer `textContent` to
  `innerHTML`.
- Users who must write HTML get a maintained sanitizer with a tag/attribute allowlist (or a small
  markup language), never a hand-written filter.
- Output encoding is the primary XSS defense; input validation is a second layer; a Content
  Security Policy is a third.
- State-changing requests need CSRF protection (token or framework equivalent) plus correct cookie
  attributes. Never change state on `GET`. More in `references/web-and-browser.md`.

## 4. Authentication [write, review]
- **Store passwords** with a slow, salted, memory-hard hash: Argon2id (preferred), scrypt, or
  bcrypt (legacy), via a maintained library that stores algorithm and parameters with the hash.
  Never MD5, SHA-1, SHA-256 alone, or a home-made scheme. Re-hash at login when parameters are
  raised. Compare with the library's verify function, not `==`.
- **Password policy:** accept long passphrases (at least 64 characters) and every Unicode
  character; do not truncate or transform the password; check against common and breached
  passwords; no composition rules; no forced periodic rotation; allow paste.
- **One generic failure message** for wrong user and wrong password, produced by **one** code
  path, with the same status code and similar response time. This also applies to registration
  ("email already taken") and password recovery. If usernames are public by design (handles
  shown on profiles), a "name taken" answer at registration is acceptable: say so explicitly and
  keep login and reset generic.
- **Throttle guessing** per account and per source, on login, password change, reset and any
  other credential check (if the entry point hides the client address, as an in-process handler
  does, throttle per account and globally, and record "no per-source limit; add it at the proxy"
  as a residual risk); slow down or lock for a short period, do not announce the lock state or
  the policy numbers, and reject attempts during lock-out **without** checking the credential.
  Make the check and the record **one atomic step** (insert the attempt, then count; or one
  transaction), or parallel guesses all pass the check. Design the lock so an outsider cannot use
  it against others: apply it to the (account, source) pair or use an increasing delay, never let a
  global counter block the correct-credential path for everyone, and do not count conflicts and
  malformed requests toward a shared quota. Cap per-user storage and row growth, and prune
  sessions and old log rows.
  Spreading one common password over many accounts is the usual bypass, so also watch per-source
  and global rates. Prefer MFA for sensitive accounts.
- **Handle failures safely:** wrap the login in catch-all handling so any unexpected error ends
  as "not authenticated" (fail closed), clears the session state and returns the generic error.
- **Credentials only over TLS, only in the body of a `POST`**; never in the URL, a cookie, a log,
  or echoed back (including in a redirect).
- **Password change:** authenticated session, current password required, new password entered
  twice, same throttling, out-of-band notification that does not contain either password.
- **Password reset:** a single-use, expiring, unguessable token from a CSPRNG, tied to one account,
  stored hashed, sent to the registered address. Build the link from configuration, never from the
  `Host` header. Same response for known and unknown accounts. Do not log the user in
  automatically, do not reveal the old password, and invalidate other sessions after the reset.
  No security questions or "hints".
- **Multistep logins:** keep progress in the server-side session, make each step check that the
  previous ones passed, never accept the username again at a later step.
- **Impersonation and "login as" features** live in internal admin tools only, with audit logging.

## 5. Sessions and tokens [write, review]
- Use the framework's session handling. A session id is a long random value from a
  cryptographically secure generator, with **no meaning**: no user data, no structure. All state
  lives on the server.
- Cookie attributes: `Secure`, `HttpOnly`, `SameSite=Lax` or `Strict`, narrowest `Path`, no
  `Domain` unless required (prefer the `__Host-` prefix). Never put the id in a URL or a log.
- **Issue a new session id at login and on every privilege change**; destroy the old one.
- **Logout and expiry on the server:** an idle timeout and an absolute timeout; logout invalidates
  the server-side session. Reject unknown ids and do not create a session from a client-supplied id.
- Sensitive actions (change email or password, payment, new payee) require recent
  re-authentication or a second confirmation.
- Self-contained tokens (JWT and similar): the server picks the allowed algorithms, rejects
  `none`, never trusts key material named in the token, and checks signature, issuer, audience and
  expiry. Keep them short-lived. Do not store tokens in `localStorage`. Details and OAuth/OIDC
  rules in `references/api-and-tokens.md`.

## 6. Access control [write, review]
- **Decide authorization on the server, for every request, in one central place** (middleware,
  decorator, policy object) that cannot be forgotten: a new route is denied until it is
  explicitly allowed. Do not copy-paste checks route by route.
- Two questions each time: may this user use this **function** (role), and may they touch this
  **object** (ownership or membership)? The second is the one that gets missed. Look up the
  object **through the caller** (`WHERE id = ? AND owner_id = ?`) or check ownership after the
  lookup, on read, update and delete alike.
- Never trust a parameter that says who or what the user is (`admin=true`, `role=`, `user_id=`);
  take identity from the session. Hiding a URL or using an unguessable id is not access control.
- **Do not assume the intended page order.** Each step of a multistep flow re-checks both
  authorization and that the earlier steps happened.
- **Mass assignment:** bind request bodies to an explicit allowlist of fields (a DTO), never
  straight onto a model, so a client cannot set `role`, `is_admin`, `owner_id`, `balance`, `price`.
- **Privilege escalation through delegation:** a user must not be able to grant more than they
  hold, or create an account more powerful than their own. Restrict who may add members or change
  roles, and check what is being granted.
- Protect static and uploaded files the same way as dynamic pages. Return `403` for "logged in
  but not allowed" and `401` for "not authenticated"; consider `404` when even the existence of
  an object is sensitive.
- Layer the rights beneath the app: separate DB accounts or at least least-privilege
  credentials, restricted OS accounts. Log every denied access and every sensitive action.

## 7. Secrets and cryptography [write, review]
- **No secrets in source, images, client code or logs.** Read them from the environment or a
  secret store. If one is committed, treat it as leaked and rotate it, not just delete it.
- **Randomness for anything security-relevant** (tokens, ids, reset codes, salts, keys) comes from
  the OS CSPRNG (`secrets`, `crypto.randomBytes`, `SecureRandom`), never `random`, `Math.random`
  or a timestamp.
- Use high-level, maintained primitives (authenticated encryption such as AES-GCM or
  ChaCha20-Poly1305, a library's sign/verify). No ECB, no static IV or nonce reuse, no MD5/SHA-1
  for security, no home-made protocols. Compare secrets and MACs in constant time.
- TLS everywhere, certificate and hostname verification **on**, never disabled "for testing".
- Client-held data that must be trusted (a price, a role) should not be sent to the client at
  all. If it has to be, sign or encrypt it **with its context** so it cannot be replayed on a
  different item or user.
- Do not log, display or prefill secrets; mask card and similar numbers.

## 8. Errors, logging and detection [write, review]
- **To the user: one generic message** and an error id. **To the log: the details.** No stack
  traces, SQL, file paths, versions or internal hostnames in responses. Turn off debug mode and
  directory listings in production. Remove sensitive comments from shipped client code.
- Catch specific errors where you can handle them; a swallowed exception on a security path is a
  fail-open bug.
- **Log security events:** logins (success and failure), password and permission changes, denied
  access, input that is overtly malicious, key business actions (payments, transfers). Include
  time, source, account and outcome. **Never log** passwords, tokens, session ids, full card
  numbers or other secrets; sanitize line breaks in logged user data.
- Protect the logs themselves from tampering and from casual reading.
- Feed the same events to alerting where you can: repeated failures, requests that tamper with
  hidden fields, bursts from one source.

## 9. Business logic and state [write, review]
- Write down each assumption the design makes about the user ("they will come from page 2",
  "the amount is positive", "this is sent once") and ask how a hostile user breaks it. They can
  skip steps, repeat, reorder, omit parameters, add parameters you did not ask for, and send
  requests in parallel.
- **Money and limits:** use exact decimal or integer minor units; reject negative, zero and
  overflowing amounts explicitly; apply discounts after the order is final; check cumulative limits
  (credit notes, refunds, transfers) against what was actually paid.
- **Do the whole check and the whole change as one atomic step.** A check followed by a separate
  update is a race (two parallel requests both pass): use a transaction, a unique constraint, a
  conditional update or a lock.
- **Validate first, then act.** Consume a code, send an email or charge a card only after
  everything that can fail has succeeded.
- A search, an export or an autocomplete must run with the caller's rights, or it leaks data
  the caller cannot open directly.
- Do not let anyone delete or edit the audit trail; a user who can create users must not be able to
  create a peer with equal powers where dual control is required.
- Sensitive operations are idempotent or protected against replay.

## 10. Files and uploads [write, review]
- Require authentication and authorization to upload; protect the endpoint against CSRF.
- Allow only the file types the business needs (extension allowlist, checked after decoding the
  name); do not trust `Content-Type`; check the real content (magic bytes) as an extra check.
- **Store with a server-generated name**, outside the web root or on a separate host, with no
  execute permission; never serve uploads from where the server would run them. Disable
  per-directory config overrides. Serve downloads with a fixed `Content-Type` and
  `Content-Disposition`, plus `X-Content-Type-Options: nosniff`.
- Limit size (also after decompression), count and rate. For images, re-encode and strip metadata
  with a patched library; scan other documents.
- Do not build a file path from the client-supplied name (section 2).

## 11. Dependencies, configuration, availability [write, review]
- Pin dependencies; run the project's audit tool (`pip-audit`, `npm audit`, OWASP Dependency-Check,
  `govulncheck`) and update vulnerable ones. Prefer few, maintained libraries.
- Secure defaults in config: debug off, no default accounts or keys, restrictive CORS, TLS only,
  minimal exposed ports and admin endpoints. Management interfaces are not on the public internet.
- Resource limits: request size, upload size, pagination caps, timeouts, rate limits, and an upper
  bound on work done per unauthenticated request. Do not let unauthenticated requests consume
  significant resources; reject excess load early.

## 12. Feature recipes [write]
For the feature you are building, make sure each line has code **and** a test (section 13).
- **Login:** generic error, throttling, constant-ish timing, hash verify, new session id, secure
  cookie, audit log, fail closed.
- **Registration:** no account enumeration, password policy, hash at rest, email confirmation
  from an unguessable single-use link.
- **Password reset:** section 4 list; token hashed, expiring, single-use; generic response.
- **Resource endpoint (`GET/PUT/DELETE /things/{id}`):** authentication, function-level check,
  object-level ownership check, DTO allowlist on write, generic 404/403, audit log.
- **List/search endpoint:** caller-scoped query, page-size cap, allowlisted sort fields.
- **Payment / transfer:** positive exact amount, server-side price, atomic check-and-update,
  idempotency key, re-authentication, audit log.
- **Upload / download:** section 10 list.
- **Fetch a URL / webhook:** SSRF rules in section 2.
- **Admin or delegation feature:** least privilege, no self-escalation, audit log, separate
  surface.

## 13. Security tests to write [write]
A security rule without a failing test is not enforced. For each rule you add, include a test
that sends the **hostile** version and expects refusal:
- another user's object id (expect 403/404) and a missing/expired session (expect 401);
- injection strings in every input (`' OR 1=1--`, `; id`, `../../etc/passwd`, `<script>`,
  `{"$ne":null}`), expecting a rejection or inert storage, never an error page;
- huge, negative, zero, non-ASCII and missing values; wrong content types; oversized bodies;
- replay, double submit and parallel requests on single-use actions;
- extra fields in the body (`role`, `is_admin`, `owner_id`) that must be ignored;
- tampered or expired token, wrong algorithm, wrong audience;
- failure in a later step leaves nothing half-done (code not consumed, no email sent);
- responses and logs contain no secret, stack trace or internal path.
Prove the test can fail: run it once against the vulnerable version or with the check removed.
For the important controls (ownership check, throttle, token expiry, path containment, role
check) do a quick **mutation check**: in a throwaway copy remove or invert the control, run the
tests, and confirm at least one fails. A control whose removal changes no test result is not
enforced; add the missing test or, if two checks overlap on purpose, note it as defense in depth.

## Reference files (load on demand)
| File | Open when |
|---|---|
| `references/finding-catalog.md` | Reviewing or auditing, or self-review of a non-trivial change (IDs and CWE numbers) |
| `references/web-and-browser.md` | HTML output, cookies, CSRF, CORS, CSP, clickjacking, headers, redirects |
| `references/api-and-tokens.md` | REST/JSON APIs, JWT, OAuth2/OIDC, API keys, rate limiting, SSRF, service-to-service calls |
| `references/attack-patterns.md` | Auditing code, searching for vulnerabilities, or when you need the attacker's view of a flaw |
