---
name: vibe-proof
description: |
  Security-focused hardening for vibe-coded full-stack apps. Runs parallel
  audits across frontend, backend, and config layers, then fixes issues
  systematically by severity. Covers injection, PII exposure, missing
  headers, error leakage, dead code, and credential hygiene.
license: MIT
---

# Vibe-proof: security hardening for full-stack apps

**Purpose**: Audit and fix security vulnerabilities in vibe-coded full-stack applications through parallel multi-layer analysis and guided remediation, fixing in severity order.

## Origin

Refined across two real hardening sessions: a React + Express + Stripe e-commerce platform deployed to Vercel, then a Next.js + Supabase + CRM medical platform. Between both sessions, 85+ issues were found, including SQL injection, hardcoded backdoor passwords, secrets in URL params, `.env` files in git, and missing security headers.

## When to use

- After vibe-coding an MVP with API routes, databases, or payment integrations
- Before a first real deployment or first real customer
- When you suspect "it works, but is it safe?"
- Any Express / React / Next.js / Nuxt app with a backend

## The seven security checks

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

## Execution process

### Phase 1: parallel audit (read-only)

Launch three scans in parallel to cover different layers simultaneously. Each can run as a dedicated audit agent if your setup has them (for example a frontend-security audit agent, a backend/API-security audit agent, and a config/credential audit agent), or as three focused passes by a single agent. The prompts below define what each pass looks for.

**Frontend audit:**

```text
Audit the frontend code for security issues:
- XSS vectors (dangerouslySetInnerHTML, unescaped user input)
- Sensitive data in client-side code
- Tracking pixels with undefined variables
- console.log statements leaking data
- Dead/unused components
- API keys or tokens in VITE_*/NEXT_PUBLIC_* env vars that should not be public
- Video embeds not using privacy-enhanced mode
- sessionStorage/localStorage holding PII unnecessarily
Report each issue with file:line, severity, and a fix suggestion.
```

**Backend / API audit:**

```text
Audit the backend code for security issues:
- SQL injection (user input in query strings, unvalidated sort/filter)
- Missing input validation on POST/PUT endpoints
- Hardcoded PII (addresses, phone numbers, names)
- Hardcoded passwords or backdoor auth strings
- API tokens or secrets in URL params instead of the Authorization header
- GET handlers that alias POST on write endpoints
- console.log/error statements (should use a structured logger)
- Error handlers leaking internal details (returning err.message to the client)
- Missing rate limiting on sensitive endpoints (including admin/sync)
- Missing enum/allowlist validation on fields like gender, status, role
- File extension derived from user filename instead of validated MIME type
- Duplicate utility code that should be extracted to shared modules
Report each issue with file:line, severity, and a fix suggestion.
```

**Config and credential audit:**

```text
Audit configuration and credentials:
- .env files tracked in git (git ls-files | grep -i env)
- .env.example missing required variables
- Security headers present/missing (check next.config headers or Express middleware)
- X-XSS-Protection should be "0" (not "1; mode=block")
- HSTS header with adequate max-age (63072000+) and preload
- CSP img-src using a wildcard https: instead of specific domains
- CSP script-src with unnecessary unsafe-eval
- localhost URLs in production allowlists (ALLOWED_ORIGINS, CSP connect-src)
- Body size limits configured (Express)
- Session configuration (secret length, cookie flags)
- Trust proxy setting
- Conflicting static + dynamic files (robots.txt vs robots.ts)
- Dead code files (unused components, disabled features)
- Unused npm dependencies (especially heavy ones like shader libs)
Report each issue with file:line, severity, and a fix suggestion.
```

### Phase 2: synthesize and prioritize

Combine all findings into one prioritized list:

| Priority | Category | Fix order |
|----------|----------|-----------|
| CRITICAL | Backdoor passwords, injection, credential leaks, secrets in URLs | Fix first |
| HIGH | PII exposure, missing validation, error leakage, missing HSTS, GET-as-POST | Fix second |
| MEDIUM | Missing rate limits, enum validation, dead code, CSP tightening | Fix third |
| LOW | Unused packages, `console.log`, config optimization | Fix last |

**Deduplication**: parallel passes find overlapping issues. Merge duplicates and keep the most detailed description.

### Phase 3: systematic fix execution

Fix in priority order. After each fix category, run `npm run build` (or the project equivalent) and verify no regressions.

**Common fix patterns:**

#### Backdoor password removal

```typescript
// BEFORE: hardcoded backdoor (CRITICAL)
const password = searchParams.get("password");
if (password !== "myapp2024") return unauthorized();

// AFTER: environment variable via Authorization header
const authHeader = request.headers.get("authorization");
const secret = authHeader?.replace(/^Bearer\s+/i, "");
if (!process.env.SYNC_SECRET || !secret || secret !== process.env.SYNC_SECRET) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```

#### Enum allowlist validation

```typescript
// Define allowlists as const arrays
const ALLOWED_GENDERS = ["male", "female", "other"] as const;
const ALLOWED_LANGUAGES = ["es", "en"] as const;

// Validate before using
if (!ALLOWED_GENDERS.includes(data.gender)) {
  return NextResponse.json({ error: "Invalid gender value" }, { status: 400 });
}
```

#### MIME-based file extension

```typescript
// BEFORE: trust the user filename (attackable)
const ext = file.name.split('.').pop();

// AFTER: derive from validated MIME type
const MIME_TO_EXT: Record<string, string> = {
  "application/pdf": "pdf",
  "image/jpeg": "jpg",
  "image/png": "png",
  "image/webp": "webp",
};
const ext = MIME_TO_EXT[file.type] || "bin";
```

#### DRY: extract shared service clients

```typescript
// BEFORE: createClient() duplicated across 3+ route files

// AFTER: shared module (e.g. src/lib/service.ts)
import { createClient, SupabaseClient } from "@supabase/supabase-js";

export function createServiceClient(): SupabaseClient | null {
  const url = process.env.SERVICE_SUPABASE_URL;
  const key = process.env.SERVICE_SUPABASE_SERVICE_KEY;
  if (!url || !key) return null;
  return createClient(url, key);
}
```

#### SQL injection (allowlist pattern)

```typescript
const ALLOWED_SORT_COLUMNS: Record<string, string> = {
  'created': 'created_at',
  'rating': 'average_rating',
  'name': 'name',
  'price': 'price::numeric',
};

if (filters?.sortBy && ALLOWED_SORT_COLUMNS[filters.sortBy]) {
  const sortColumn = ALLOWED_SORT_COLUMNS[filters.sortBy];
  query += ` ORDER BY ${sortColumn}`;
}
```

#### API token in header (not URL)

```typescript
// BEFORE: token in URL (visible in logs, browser history)
const url = `${API_URL}?access_token=${TOKEN}`;

// AFTER: token in Authorization header
fetch(API_URL, {
  headers: { 'Authorization': `Bearer ${TOKEN}` }
});
```

#### Security headers, Next.js (next.config.ts)

```typescript
// In the next.config.ts headers() array:
{ key: "X-Content-Type-Options", value: "nosniff" },
{ key: "X-Frame-Options", value: "DENY" },
{ key: "X-XSS-Protection", value: "0" },  // modern: disable the buggy filter
{ key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
{ key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains; preload" },
{ key: "X-DNS-Prefetch-Control", value: "off" },
```

#### Security headers, Express (middleware)

```typescript
app.use((_req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '0');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('Strict-Transport-Security', 'max-age=63072000; includeSubDomains; preload');
  next();
});
```

#### Error response masking

```typescript
// Express
app.use((err, _req, res, _next) => {
  const status = err.status || 500;
  res.status(status).json({
    error: status >= 500 ? 'Internal Server Error' : err.message
  });
});

// Next.js route handler
} catch (err) {
  console.error("Route error:", err);  // log internally
  return NextResponse.json(
    { error: "Internal server error" },  // mask externally
    { status: 500 }
  );
}
```

#### Remove GET-as-POST alias

```typescript
// BEFORE: exposes a write endpoint to GET requests (CSRF, caching, logging)
export async function GET(request: NextRequest) {
  return POST(request);
}

// AFTER: delete the GET export entirely. Only export POST.
```

### Phase 4: credential remediation

If `.env` files were tracked in git:

```bash
# Add to .gitignore
echo ".env.production" >> .gitignore

# Remove from tracking (keeps the local file)
git rm --cached .env.production

# Then rotate ALL exposed credentials
```

**Credentials that MUST be rotated if exposed:**

- Database passwords / Supabase service role keys
- API keys (Stripe, Shippo, Resend, CRM, etc.)
- Session/sync secrets
- Webhook signing secrets

### Phase 5: environment variable provisioning

If deploying to Vercel, set env vars via the API:

```bash
# Get the project id
curl -s -H "Authorization: Bearer $VERCEL_TOKEN" \
  "https://api.vercel.com/v9/projects?teamId=$TEAM_ID" | \
  jq '.projects[] | select(.name == "PROJECT_NAME") | .id'

# Set an env var (returns ENV_CONFLICT if it already exists)
curl -s -X POST \
  "https://api.vercel.com/v10/projects/$PROJECT_ID/env?teamId=$TEAM_ID" \
  -H "Authorization: Bearer $VERCEL_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"key": "VAR_NAME", "value": "VAR_VALUE", "type": "encrypted", "target": ["production", "preview"]}'
```

### Phase 6: verify and deploy

```bash
# Build verification
npm run build

# Commit (conventional commit format)
git commit -m "fix(security): harden platform [summary of fixes]"

# Push and deploy
git push origin main
```

### Phase 7: post-deploy connection verification

After deploy, verify all external services are reachable from production:

1. Test each service endpoint with a minimal query (Supabase: `select count`, CRM API: `GET /contacts?limit=1`).
2. Check for common failures:
   - `ENOTFOUND` / `NXDOMAIN` means a Supabase free-tier project is paused (unpause in the dashboard).
   - `401 Unauthorized` means an API key mismatch between `.env.local` and the deploy env vars.
   - A `schema cache` error means Supabase just restored; wait 1-2 minutes.
3. Clean up any test records created during verification (check all synced downstream services).

## Success criteria

- [ ] Build passes with zero warnings
- [ ] No user input reaches SQL without parameterization or allowlist
- [ ] No PII or hardcoded passwords in source code
- [ ] No API tokens or secrets in URLs
- [ ] No `.env` files tracked in git
- [ ] Security headers present on all responses (HSTS, DENY, nosniff)
- [ ] Error responses do not leak internals
- [ ] All POST/PUT endpoints validate input (including enum allowlists)
- [ ] File extensions derived from MIME type, not filename
- [ ] Dead code deleted, not commented
- [ ] No GET aliases for POST endpoints
- [ ] Exposed credentials rotated
- [ ] All external service connections verified post-deploy

## Lessons learned

1. **Parallel audit passes save time.** Three passes scanning different layers at once catch issues that sequential review misses.

2. **Sort columns are the number-one SQL injection vector in vibe-coded apps.** Everyone parameterizes WHERE clauses and forgets ORDER BY.

3. **`.env` files in git are common.** Always check `git ls-files | grep -i env` as the very first step.

4. **Hardcoded "temporary" passwords become permanent backdoors.** Search for string comparisons against literals in auth logic: `password !== "something"`, `secret === "hardcoded"`.

5. **GET aliasing POST is a silent CSRF vector.** `export { POST as GET }` or `GET(req) { return POST(req) }` exposes write endpoints to CSRF, browser prefetch, and CDN caching.

6. **`X-XSS-Protection: 1; mode=block` is outdated.** The modern recommendation is `0`; the browser filter itself has been exploited. CSP is the real protection.

7. **Enum validation is easy to forget.** Gender, status, role, language: any field with a finite set of values needs a `const` allowlist, not just type checking.

8. **File extension from filename is attackable.** A file named `malware.pdf.exe` with an `application/pdf` MIME type should get `.pdf` from the MIME type, not `.exe` from the name.

9. **Duplicate service client code means duplicate security gaps.** Extract shared modules (`lib/service.ts`, `lib/crm.ts`) so auth logic is fixed in one place.

10. **The Vercel API for env vars is faster than the dashboard.** One curl loop sets ten variables in seconds.

11. **Supabase free tier pauses after inactivity.** The DNS record disappears (NXDOMAIN). After unpause, the schema cache takes 1-2 minutes to warm. Production may work before local because of separate connection pools.

12. **Always test connections after deploy.** A passing build does not prove services are reachable. Test each external service with a minimal query post-deploy.

13. **`localhost` in production origin allowlists is a real finding.** It is easy to leave `http://localhost:3000` in `ALLOWED_ORIGINS` during development.

---

See full content and updates at https://github.com/HermeticOrmus/vibe-proof-skills.
