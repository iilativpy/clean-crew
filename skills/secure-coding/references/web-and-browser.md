# Web and browser security

Open when the change renders HTML, sets cookies, accepts cross-site requests, adds CORS, a
CSP, a redirect, or response headers. Rules are written as defaults; frameworks often cover
some of them, so check what the framework already does before adding code.

## The browser's trust model, briefly
- The **same-origin policy** (scheme + host + port) stops one site's script from reading
  another's responses. It does **not** stop one site from *sending* requests to another (that
  is what CSRF abuses), and cookies go along automatically with those requests.
- Cookies are scoped by **site and path**, not by port, and a sibling subdomain can set cookies
  for the parent. Treat every subdomain of the cookie's domain as able to attack the app.
- Anything the browser renders from your origin runs with that origin's rights. Hence XSS is
  game over for every other browser-side defense (CSRF tokens, session protection).

## Output encoding by context
| Where the data lands | What to do |
|---|---|
| HTML body | HTML-encode `& < > " '`, or set `textContent` |
| Quoted attribute | Attribute-encode and always quote; keep attribute names fixed |
| `href`/`src`/`action` | Allow only `http`/`https` (or relative); then attribute-encode |
| JavaScript string | Avoid. If unavoidable, put JSON in a data attribute or a non-executable `<script type="application/json">` and parse it |
| URL parameter | `encodeURIComponent`-style percent-encoding of the value only |
| CSS | Do not interpolate; set a property from an allowlist of values |
| Inside `<script>` code, event handlers, `style`, comments, tag names | Never |
- Encode **every** character that could matter, not only the ones the current context seems to
  need: contexts get switched by tolerant browsers and by a second field.
- Set the response charset explicitly and do not let a parameter choose it.
- DOM XSS: do not feed `location`, `document.referrer`, `postMessage` data or storage to
  `innerHTML`, `document.write`, `eval` or `setTimeout(string)`. Use safe sinks.
- Trusted Types (`require-trusted-types-for 'script'`) and a nonce-based CSP make the remaining
  mistakes harder to exploit; they do not replace encoding.

## CSRF
- Use the framework's built-in protection first. Otherwise:
  - **Synchronizer token**: a per-session secret, unpredictable, sent in a hidden field or header
    (not in a cookie, a URL or a log), checked on every state-changing request.
  - **Stateless apps**: signed double-submit (HMAC over the session-bound value with a server
    secret). A plain double-submit cookie is bypassable by cookie injection.
  - Add **Origin/`Sec-Fetch-Site`** checks as a second layer: reject cross-site state-changing
    requests; compare the full origin, not a prefix; treat a missing or `null` origin as a failure
    unless another check passes.
  - For JSON APIs that use cookies: require `Content-Type: application/json`, reject form types, and
    require a custom header with a strict CORS allowlist.
- `SameSite=Lax/Strict` is a good extra layer, but it is scoped to the **site** (sibling
  subdomains are not blocked) and does not replace a token.
- Never change state on `GET`/`HEAD`.
- **Login CSRF**: use a token on the login form and issue a fresh session after login.
- Tokens must be tied to the user's session; a token that works for any session defeats the point.
- Per-request or per-page tokens are stronger but break the back button; use them for the most
  sensitive actions.

## Cookies
`Secure; HttpOnly; SameSite=Lax` (or `Strict`), narrow `Path`, no `Domain`, `__Host-` prefix when
possible, short lifetime, not persistent unless needed. `SameSite=None` requires `Secure`.

## CORS
- Disabled unless needed. Allow an **explicit list** of origins; never echo the request `Origin`
  back blindly; never combine `*` with credentials; do not trust the `null` origin.
- CORS is a browser rule that relaxes the same-origin policy. It is not authentication and does
  not stop non-browser clients.

## Content Security Policy and other headers
- Prefer a strict, nonce- or hash-based CSP, with a fresh nonce for each response:
  `script-src 'nonce-<random>' 'strict-dynamic'; object-src 'none'; base-uri 'none'`.
  Allowlist-based CSPs are easier to bypass. Roll out in report-only mode first.
- Clickjacking: `Content-Security-Policy: frame-ancestors 'none'` (or `'self'`), plus the older
  `X-Frame-Options: DENY` for legacy browsers.
- `X-Content-Type-Options: nosniff`; `Strict-Transport-Security` with a long max-age;
  `Referrer-Policy: no-referrer` or `strict-origin-when-cross-origin` (use `no-referrer` on pages
  carrying secrets in the URL, such as reset links); `Cache-Control: no-store` on sensitive
  responses.
- Disable `Server`/version banners; remove debug and sourcemap files from production.

## Redirects and links
- Never redirect to a user-supplied URL; use an index into a list of allowed targets.
- If a return path is required, accept only a relative path beginning with a single `/` followed by
  a letter, with no scheme and no `//` or `\`. Reject everything else, do not sanitize.
- Add `rel="noopener noreferrer"` to links that open untrusted pages in a new tab.

## Data stored in the browser
- Do not keep session ids, tokens or secrets in `localStorage`/`sessionStorage` (any XSS reads
  them). Do not put secrets in cacheable pages or autocomplete fields.
- Client-side scripts, hidden fields and `maxlength` are conveniences; the server re-validates all.
- Do not hide anything sensitive in JavaScript, comments or mobile/browser extension code; it can
  be read.

## Local checks (quick tell-tales)
`innerHTML =`, `document.write`, `eval(`, `|safe`, `v-html`, `dangerouslySetInnerHTML`, `href={user}`,
`res.redirect(req.`, `Access-Control-Allow-Origin` set from the request, cookies set without flags,
forms that POST without a token field, `GET` routes that delete or update.
