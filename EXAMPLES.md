# Examples

Concrete before/after for the fix patterns this skill applies. Each shows the vibe-coded version that ships by default and the hardened version that replaces it.

---

## Backdoor password removal

**Severity: CRITICAL.** A "temporary" password compared against a literal in auth logic becomes a permanent backdoor. Search for string comparisons against literals: `password !== "something"`, `secret === "hardcoded"`.

```typescript
// BEFORE: hardcoded backdoor
const password = searchParams.get("password");
if (password !== "myapp2024") return unauthorized();

// AFTER: environment variable via Authorization header
const authHeader = request.headers.get("authorization");
const secret = authHeader?.replace(/^Bearer\s+/i, "");
if (!process.env.SYNC_SECRET || !secret || secret !== process.env.SYNC_SECRET) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```

---

## Enum allowlist validation

**Severity: HIGH.** Any field with a finite set of values (gender, status, role, language) needs a `const` allowlist, not just a type check. Type checking confirms it is a string; it does not confirm it is an allowed string.

```typescript
// Define allowlists as const arrays
const ALLOWED_GENDERS = ["male", "female", "other"] as const;
const ALLOWED_LANGUAGES = ["es", "en"] as const;

// Validate before using
if (!ALLOWED_GENDERS.includes(data.gender)) {
  return NextResponse.json({ error: "Invalid gender value" }, { status: 400 });
}
```

---

## MIME-based file extension

**Severity: HIGH.** A file named `malware.pdf.exe` with an `application/pdf` MIME type should get a `.pdf` extension from the validated MIME type, not `.exe` from the user-supplied name.

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

---

## SQL injection: sort-column allowlist

**Severity: CRITICAL.** Sort columns are the most common SQL injection vector in vibe-coded apps. Everyone parameterizes the `WHERE` clause and forgets the `ORDER BY`. A column name cannot be parameterized, so it must be mapped through an allowlist.

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

---

## Token in header, not URL

**Severity: CRITICAL.** A token in a URL leaks into server logs, browser history, and `Referer` headers. Move it into the `Authorization` header.

```typescript
// BEFORE: token in URL (visible in logs, browser history)
const url = `${API_URL}?access_token=${TOKEN}`;

// AFTER: token in Authorization header
fetch(API_URL, {
  headers: { 'Authorization': `Bearer ${TOKEN}` }
});
```

---

## Security headers

**Severity: HIGH.** Missing headers leave the app open to clickjacking, MIME sniffing, and downgrade attacks. Add them once at the framework boundary.

Next.js, in the `next.config.ts` `headers()` array:

```typescript
{ key: "X-Content-Type-Options", value: "nosniff" },
{ key: "X-Frame-Options", value: "DENY" },
{ key: "X-XSS-Protection", value: "0" },  // modern: disable the buggy filter
{ key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
{ key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains; preload" },
{ key: "X-DNS-Prefetch-Control", value: "off" },
```

Express, as middleware:

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

`X-XSS-Protection: 1; mode=block` is outdated. The modern recommendation is `0`, because the browser filter itself has been exploited. CSP is the real protection.

---

## Error response masking

**Severity: HIGH.** A 500 that returns `err.message` hands an attacker stack traces, file paths, and library versions. Log the detail internally, return a generic message externally.

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

---

## Remove GET-as-POST alias

**Severity: HIGH.** Aliasing a write endpoint to GET exposes it to CSRF, browser prefetch, and CDN caching. A GET should never mutate state.

```typescript
// BEFORE: exposes a write endpoint to GET requests
export async function GET(request: NextRequest) {
  return POST(request);
}

// AFTER: delete the GET export entirely. Only export POST.
```

---

## DRY: extract shared service clients

**Severity: MEDIUM.** When `createClient()` is duplicated across route files, a security fix to the auth logic has to be applied in every copy, and one will be missed. Extract a shared module so the auth logic lives in one place.

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

---

## A note on the audit

These fixes come out of two real hardening sessions: a React + Express + Stripe e-commerce platform, and a Next.js + Supabase + CRM medical platform. Between them, 85+ issues surfaced. The pattern that repeated most: the app worked, the demo was clean, and the security holes were invisible until something scanned for them deliberately. A passing build is not a security audit.

## Further reading

- [`mars-skills`](https://github.com/HermeticOrmus/mars-skills): the broader production-readiness companion
- [`vibe-engineer-skills`](https://github.com/HermeticOrmus/vibe-engineer-skills): the discipline of directing AI codegen well
