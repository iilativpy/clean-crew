# Attacker's view and code signatures

Open when auditing code, hunting for vulnerabilities, or when you need to know how a flaw is
actually exploited. Each entry: **what the attacker does**, **what the vulnerable code looks like**,
**what to grep for**, **the fix** (and the catalog ID). Greps find candidates; a finding exists
only after you trace the value from a source to the sink and see no effective control.

## How to hunt
1. **Sources** (where outside data enters): request parameters, body, headers, cookies, path
   segments, uploaded files and names, webhooks, queue messages, rows another user wrote, files
   read from disk, environment of other services.
2. **Sinks** (where it becomes dangerous): query builders, process launchers, file APIs, template
   and HTML output, redirects, HTTP clients, deserializers, `eval`, header setters, logs, crypto and
   token verifiers.
3. **Controls on the path**: validation, encoding, parameter binding, authorization checks.
4. Trace both ways. Start from sinks (grep) for breadth, and from each entry point for depth.
   Data that passes through several functions changes shape; read the whole route.
5. Check **absent** controls too: a route with no authorization call, a write with no CSRF token, a
   login with no throttle. Missing code cannot be grepped, so list the routes and compare.

## Language-neutral signatures
| Flaw | Look for |
|---|---|
| SQL injection (SEC-INJ-1) | quoted SQL keywords concatenated or formatted with variables: `"SELECT `, `"INSERT `, `"UPDATE `, `"DELETE `, `" WHERE `, `" AND `, `" ORDER BY `, f-strings, `%`, `.format`, `+`, template literals, `String.format` building queries; raw-query escape hatches of an ORM (`raw`, `execute`, `query`, `$queryRawUnsafe`) |
| Command injection (SEC-INJ-3) | `os.system`, `subprocess` with `shell=True`, `popen`, `exec`, `child_process.exec`, backticks, `Runtime.exec` with a single string, `system(` |
| Dynamic code (SEC-INJ-4) | `eval`, `exec`, `compile`, `new Function`, `setTimeout("...")`, `include`/`require` with a variable, `render_template_string`, `Template(user_text)` |
| Path traversal (SEC-INJ-5) | file APIs (`open`, `readFile`, `sendFile`, `File(`, `Paths.get`, `os.path.join`) with a request-derived part and no containment check; ZIP extraction without checking entry names (zip-slip) |
| XXE (SEC-INJ-6) | XML parser creation without disabling DTD/external entities |
| Deserialization (SEC-INJ-7) | `pickle.load(s)`, `yaml.load(` without safe loader, `ObjectInputStream`, `readObject`, `BinaryFormatter`, `unserialize`, `Marshal.load`, JSON libs with type names enabled |
| SSRF (SEC-FIL-4) | `requests.get(url)`, `fetch(url)`, `HttpClient`, `urlopen`, `curl` where `url` comes from a request, webhook config, image/URL import, PDF/HTML renderers |
| Open redirect (SEC-OUT-5) | `redirect(request...)`, `Location` header, `res.redirect(req.query.next)`, `sendRedirect` with a parameter |
| XSS (SEC-OUT-1/2) | HTML assembled by concatenation, `innerHTML`, `document.write`, `|safe`, `mark_safe`, `Markup(`, `v-html`, `dangerouslySetInnerHTML`, `raw`, `<%-`, unescaped template tags |
| Weak password storage (SEC-AUTH-1) | `md5`, `sha1`, `sha256` applied to a password, `hashlib` near "password", custom salting, base64/encryption of passwords |
| Weak randomness (SEC-SES-1, SEC-CRY-3) | `random.`, `Math.random`, `rand(`, time-based ids or tokens used for sessions, resets, keys |
| Secrets (SEC-CRY-1) | `password =`, `secret`, `api_key`, `token`, `BEGIN PRIVATE KEY`, `AKIA`, `.env` files in the repo, credentials in test fixtures that are also used in production, config with real hostnames |
| TLS off (SEC-CRY-4) | `verify=False`, `rejectUnauthorized: false`, `InsecureSkipVerify`, trust-all managers, `curl -k` |
| JWT misuse (SEC-SES-5) | `verify=False`, `algorithms` missing or including `none`, `decode` without verification, `jwt.decode(..., options={"verify_signature": False})`, key from header |
| Debug/verbose errors (SEC-ERR-1) | `DEBUG = True`, `app.run(debug=True)`, error handlers that return `str(e)`/traceback, `printStackTrace` to the response |
| Mass assignment (SEC-ACC-3) | `Model(**request.json)`, `update(request.body)`, `Object.assign(model, req.body)`, `@RequestBody` entities bound directly |
| Missing authz (SEC-ACC-1/2/5) | route handlers with no decorator/middleware/policy call; queries by `id` alone; admin routes distinguished only by URL |
| CSRF (SEC-WEB-1/2) | cookie-authenticated `POST/PUT/DELETE` without token or Origin check; state change on `GET` |
| Cookie flags (SEC-WEB-3) | `set_cookie(`, `res.cookie(`, `Set-Cookie` without `Secure`/`HttpOnly`/`SameSite` |
| CORS (SEC-WEB-4) | `Access-Control-Allow-Origin` reflecting the request origin, `*` with credentials |
| Log secrets (SEC-ERR-2) | `log`/`print` of request bodies, headers, `Authorization`, passwords, tokens |
| Swallowed errors (SEC-ERR-4) | bare `except:`, `except Exception: pass`, empty `catch`, `return true` in a catch around an auth check |
| Race (SEC-LOG-2) | read-modify-write on balance, stock, counters or single-use codes with no transaction, lock or unique constraint |

## Attack stories (to recognize the shape of a flaw)
- **Account takeover through enumeration plus weak reset.** Attacker learns which emails
  exist from different messages, then guesses or reuses a reset token that is short, reusable or
  never expires. Fix: generic responses, long random single-use expiring tokens (SEC-AUTH-3, -7).
- **Horizontal privilege escalation (IDOR).** Attacker changes `/orders/1042` to `/orders/1043`
  and reads someone else's order because the handler only checks that *a* user is logged in.
  Fix: owner-scoped lookup on every object access (SEC-ACC-1).
- **Mass assignment.** Attacker adds `"role":"admin"` to a profile update and the model binder
  saves it. Fix: DTO allowlist (SEC-ACC-3).
- **Second-order injection.** A name with a quote is stored safely, then pasted into a query
  by another feature (report, export, admin page) that trusts "our own data". Fix: bind every
  value, always (SEC-INJ-1).
- **Stored XSS to session theft or action.** A comment containing script runs in every visitor's
  browser and calls the API with their cookies. Fix: output encoding; cookie flags and CSP only
  limit the damage (SEC-OUT-1).
- **Filter bypass.** Blocklists fall to `SeLeCt`, `/**/`, `%2527`, `<scr<script>ipt>`, NUL bytes,
  full-width or other Unicode forms. Fix: allowlists, binding, encoding, reject (section 1).
- **Client-trusted price or role.** A hidden field `price=1` or `admin=true` is accepted.
  Fix: look prices and roles up on the server (SEC-LOG-1, SEC-ACC-4).
- **Parallel requests.** Two simultaneous redeem requests both see "unused" and both succeed.
  Fix: atomic conditional update or unique constraint (SEC-LOG-2).
- **Privilege escalation by delegation.** A read-only member adds a new account with full rights
  because "add member" only requires read access. Fix: require at least the rights you grant,
  or restrict who may add members (SEC-ACC-6).
- **Forced browsing.** The "edit user" page is hidden from the menu but reachable by URL, or the
  last step of a checkout can be called without the payment step. Fix: central deny-by-default
  checks and per-step state (SEC-ACC-2, -8).
- **SSRF to cloud metadata.** A "fetch image from URL" feature is pointed at
  `http://169.254.169.254/` and returns credentials. Fix: IP and scheme validation, no redirects,
  metadata protection (SEC-FIL-4).
- **Logic flaw: negative or huge numbers.** A refund of `-50` credits the attacker, a quantity of
  `99999999999999999999` crashes the server or wraps an integer. Fix: explicit range checks and
  exact arithmetic (SEC-LOG-1, SEC-ERR-7).
- **Information leak by error.** A malformed request returns the SQL, path and framework
  version that guide the next attack. Fix: generic errors (SEC-ERR-1).

## Probing hostile input safely (in a throwaway copy)
Send, to every input of the thing you run: empty, whitespace, very long (10k+ characters), huge
numbers (30+ digits), negative and zero, `NaN`/`Infinity`, non-ASCII digits and letters, NUL
byte, newline and CR in a header-like field, `' " ; | & $ ` < > { } ../`, wrong types (string for
number, array for string, object for scalar), duplicate parameters, missing parameters, extra
parameters, wrong content type, an id that belongs to another user, an expired/forged token.
Record: crash or traceback, different status for "exists" and "does not exist", reflected input,
accepted extra fields, differences in timing. Never run probes against a system you do not own or
have not been told to test.
