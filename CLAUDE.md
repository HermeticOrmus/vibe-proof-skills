# CLAUDE.md

A security-hardening protocol for the **vibe-coded full-stack app**: the MVP that works but hasn't been checked for injection, leaked secrets, or missing headers. Drop this file at the root of your repository and ask Claude to harden the project. Merge with project-specific instructions as needed.

**Tradeoff**: This protocol biases toward auditing before shipping. Run it before a first real deployment or first real customer, not on every commit.

## How to use this file

Ask Claude: "security audit this project against CLAUDE.md, then fix by severity." Claude audits the three layers (frontend, backend, config) against the seven checks below, synthesizes findings into one prioritized list, and fixes in severity order with a build after each category.

## The seven security checks

Run every box. Each unchecked box is a finding.

### 1. Injection vectors

- [ ] No user input in SQL/query strings without parameterization
- [ ] Sort columns and filter fields use allowlist validation
- [ ] No `eval()`, `new Function()`, or template-literal injection
- [ ] URL params parsed with bounds checking (`parseInt` with min/max)
- [ ] Enum fields (gender, status, role) validated against `const` allowlists

### 2. PII and secret exposure

- [ ] No hardcoded addresses, phone numbers, or names in source
- [ ] No hardcoded passwords or backdoor auth strings
- [ ] API tokens in headers (`Authorization`), never in URL params
- [ ] Admin endpoint secrets use `Authorization: Bearer`, not query params
- [ ] No `.env` files tracked in git (`git ls-files | grep -i env`)
- [ ] No secrets in client-side code or in `VITE_*` / `NEXT_PUBLIC_*` vars that should not be public
- [ ] `.env.example` documents all required variables (sync secrets, CRM keys, service keys)
- [ ] No `localhost` URLs in production allowlists (`ALLOWED_ORIGINS`, CSP, etc.)

### 3. Missing security headers

- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options: DENY` (or `SAMEORIGIN` if iframes are needed)
- [ ] `X-XSS-Protection: 0` (modern best practice; disables the buggy browser filter)
- [ ] `Referrer-Policy: strict-origin-when-cross-origin`
- [ ] `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
- [ ] `X-DNS-Prefetch-Control: off` (privacy; prevents browser DNS leaks)
- [ ] Body size limits on `express.json()` and `express.urlencoded()` (Express)
- [ ] CSP `img-src` restricted to specific CDN domains, not an `https:` wildcard
- [ ] CSP `script-src` without `unsafe-eval` (remove if WebGL/shaders were deleted)

### 4. Error leakage

- [ ] Production error responses do not expose stack traces
- [ ] 500 errors return a generic message, not `error.message`
- [ ] No `console.log` of sensitive data (tokens, passwords, PII)
- [ ] A structured logger is used instead of `console.*` in production code
- [ ] Catch blocks return masked errors: `"Internal server error"`, not `err.message`

### 5. Input validation gaps

- [ ] All POST/PUT endpoints validate the body with Zod or equivalent
- [ ] Query params have type coercion and bounds (`limit`, `offset`, `id`)
- [ ] Integer params checked against `MAX_INT` (2147483647)
- [ ] Enum params validated against `const ALLOWED_X = [...] as const` allowlists
- [ ] File uploads check size AND validate magic bytes, not just the MIME header
- [ ] File extensions derived from validated MIME type, not the user-supplied filename
- [ ] Token/secret params validated for format (min length, charset) before DB lookup
- [ ] Text inputs sanitized (strip HTML tags, dangerous chars) before storage

### 6. Dead code and attack surface

- [ ] Unused routes/endpoints removed
- [ ] Unused components deleted, not commented out
- [ ] Disabled features removed entirely, not just `if (false)`
- [ ] Test/debug endpoints not present in production
- [ ] Unused npm packages removed
- [ ] No GET handler aliasing POST on write endpoints (`export { POST as GET }`)
- [ ] No conflicting static + dynamic files (e.g. `robots.txt` + `robots.ts`)
- [ ] Unused client utility functions removed (dead `createBrowserClient`, etc.)
- [ ] Video embeds use privacy-enhanced mode (`youtube-nocookie.com`)

### 7. Credential hygiene

- [ ] Session secrets are 32+ characters
- [ ] Cookies set `httpOnly`, `secure` (production), `sameSite: 'lax'`
- [ ] Trust proxy configured when behind a reverse proxy (Vercel, nginx)
- [ ] Webhook endpoints verify signatures (Stripe, etc.)
- [ ] Rate limiting on auth, checkout, newsletter, AND admin/sync endpoints
- [ ] Rate-limiting strategy fits the platform (in-memory is defense-in-depth on serverless; use Upstash/KV for persistent limiting)

## Fix order by severity

Audit findings from all three layers merge into one list, then fix in this order:

| Priority | Category | Fix order |
|----------|----------|-----------|
| CRITICAL | Backdoor passwords, injection, credential leaks, secrets in URLs | Fix first |
| HIGH | PII exposure, missing validation, error leakage, missing HSTS, GET-as-POST | Fix second |
| MEDIUM | Missing rate limits, enum validation, dead code, CSP tightening | Fix third |
| LOW | Unused packages, `console.log`, config optimization | Fix last |

After each category: run `npm run build` (or the project equivalent) and verify no regressions. If `.env` files were tracked in git, rotate every exposed credential after removing them from tracking.

---

## Trigger this protocol when

- A vibe-coded MVP is about to ship to a first real deployment or customer
- The app has API routes, a database, or a payment integration that were never security-reviewed
- The question is "it works, but is it safe?"
- Any Express / React / Next.js / Nuxt app with a backend

---

**Source**: Packaged from a security-hardening skill refined across two real sessions on full-stack apps (a React + Express + Stripe e-commerce platform and a Next.js + Supabase + CRM medical platform). Between them, 85+ issues were found, including SQL injection, hardcoded backdoor passwords, secrets in URL params, `.env` files in git, and missing security headers.

**License**: MIT. Use it, fork it, merge it into your own.
