Review the code in this project (or the specific files/changes indicated by the user: $ARGUMENTS) using the engineering best practices below. For each issue found, cite the specific rule, explain why it matters, show the problematic code, and provide the corrected version.

---

## Review Checklist

Work through EVERY section below. For each, check the relevant code and report findings. If a section doesn't apply, skip it silently.

### 1. Input Validation & Never Trust the Client
- All validation MUST happen server-side. Frontend validation is UX only.
- Return clear, field-level error messages (not vague "Validation error").
- Use schema validation (Zod, Joi, etc.) to validate types, lengths, ranges.
- Check: string lengths, array sizes, data types, file MIME types server-side.
- NEVER do `Model.create(req.body)` — always whitelist/pick accepted fields (mass assignment protection).
- Never accept `role`, `credits`, `isAdmin` or similar sensitive fields from client input.

### 2. Error Handling — UX vs Security
- Expected errors (validation, 404, 403): return friendly messages to client.
- Unexpected errors (crashes): return generic "Something went wrong" to client. Full details go to logging/monitoring only.
- NEVER expose stack traces, DB column names, file paths, or internal structure to the client in production.
- Must have a global error handler as the last line of defense.
- Error monitoring (Bugsnag/Sentry) should be set up — know about crashes before users do.

### 3. Auth vs Authorization & IDOR
- Authentication (who are you) != Authorization (are you allowed).
- Every query on user-owned data MUST include `userId` (and `tenantId` in SaaS/multi-tenant).
- Fetching by ID alone with no ownership check = IDOR vulnerability.
- For sub-resources, JOIN up to the parent and verify ownership there.
- Return 404 (not 403) when a resource doesn't belong to the user — don't confirm existence.
- Generic auth errors: never say "email not found" or "wrong password" — always "Invalid credentials".

### 4. SQL Indexes & Query Performance
- Add indexes from day one — don't wait for performance problems.
- Index every column used in WHERE, JOIN, ORDER BY clauses.
- Use UNIQUE indexes for fields that must be unique (email, username, slug, API key) — correctness AND speed.
- Understand selectivity: boolean/status columns alone have poor selectivity — combine in composite indexes.
- Composite index column order matters — put high-selectivity columns first.
- Verify with EXPLAIN ANALYZE — look for "Index Scan" not "Seq Scan".
- Don't over-index: each index slows INSERT/UPDATE/DELETE.

### 5. The N+1 Query Problem
- Never query inside a loop. If you see the same query repeated with different IDs, that's N+1.
- Fix with: JOIN, eager loading (Sequelize `include`, Mongoose `populate`), or raw SQL.
- Select only needed fields in includes/populates — don't fetch entire related records.
- Enable SQL query logging to catch N+1 patterns before merging.

### 6. Pagination — Never Load Everything
- No endpoint should ever return unbounded rows.
- Always enforce a hard server-side LIMIT cap, even if the client doesn't request pagination.
- Prefer cursor-based pagination over OFFSET (OFFSET scans and discards rows — gets slower with depth).
- Validate and cap the `limit` parameter server-side: `Math.min(Number(limit), 50)`.

### 7. Resource Limits & Rate Limiting
- Credit/quota checks MUST be server-side. Frontend checks are bypassable.
- Rate limit sensitive endpoints: login, register, OTP, forgot-password.
- Layer rate limiting: edge (Cloudflare/AWS WAF) + application (express-rate-limit).
- Without rate limiting, attackers can brute-force 100K passwords/min.

### 8. Duplicate API Calls
- POST/mutation endpoints: use loading states, disable buttons during in-flight requests.
- useEffect: use AbortController cleanup to cancel stale requests on re-render/unmount.
- Consider React Query / SWR for automatic deduplication, caching, and loading states.
- Watch for useEffect dependency arrays that cause excessive re-firing.

### 9. Return Only What's Needed
- Never expose: password, email, phone, stripeCustomerId, resetToken, role, permissions, internalNotes in public API responses.
- Always use `attributes` (Sequelize) or field selection (Mongoose `select`) to pick only needed fields.
- Different response shapes for owner vs public viewer (owner sees email/phone, public sees name/avatar only).
- Default to returning minimum. Add fields only when there's a specific need.

### 10. Password Storage & Secrets
- NEVER plain text or MD5/SHA1. Always bcrypt with minimum cost 12.
- NEVER hardcode secrets (API keys, DB passwords, JWT secrets) in source code.
- All secrets must come from environment variables. `.env` must be in `.gitignore`.
- If a secret was ever pushed to git, it is compromised FOREVER — rotate immediately.

### 11. Race Conditions
- Credit deduction, inventory, reservations: use atomic SQL updates (`SET credits = credits - cost WHERE credits >= cost`) or SELECT FOR UPDATE with transactions.
- Never read-check-write in separate steps without a lock — two concurrent requests will both pass the check.
- Watch for: overselling inventory, double-claimed referral bonuses, duplicate username registration, double-booked slots.

### 12. Parallel Execution
- If operation B doesn't need A's result, run them in parallel with `Promise.all()`.
- Sequential independent calls waste time: 5 x 100ms = 500ms sequential vs ~100ms parallel.
- Use `Promise.allSettled()` when partial results are acceptable and some calls may fail.

### 13. File Upload Security
- NEVER store uploads on the application server. Use S3, DO Spaces, BunnyCDN, Cloudflare R2.
- NEVER trust the client filename — generate a safe UUID-based name server-side.
- Validate MIME type server-side (not just extension). Enforce file size limits.
- Never serve uploaded files from the same domain as your app (XSS via uploaded HTML/SVG).

### 14. Production Architecture
- App server should never be directly exposed to the internet.
- Use proxy layer: Cloudflare (DNS + DDoS + WAF + CDN) -> ALB/Nginx -> App Server (private) -> DB (private).
- Node.js should bind to 127.0.0.1, not 0.0.0.0.
- Database must NEVER have a public IP — private subnet only.
- Security headers: X-Frame-Options, X-Content-Type-Options, Referrer-Policy, CSP, HSTS.

### 15. Logging & Monitoring
- console.log is not monitoring. Use Bugsnag, Sentry, or Datadog.
- Add user context to errors (who was affected).
- Connect to Slack for instant alerts.
- You should know about production crashes before your users do.

### 16. Scale Readiness
- Ask: what happens with 10x data, 100x users?
- Watch for: memory leaks, CPU spikes, N+1 queries, unbounded queries, connection pool limits.
- Test with realistic volume, not toy data.
- Consider bandwidth, concurrency, and connection pool limits.

### 17. Design Patterns & Architecture
- Follow established project patterns (MVC, Service Layer, Repository, etc.).
- Backend: Controller -> Service -> Repository layered architecture.
- Frontend/React: Component-based, Container/Presentational split, Custom Hooks for reusable logic.
- Prefer built-in framework features over custom solutions. Custom code = custom bugs.
- DRY / KISS / YAGNI — Don't Repeat, Keep It Simple, You Ain't Gonna Need It.
- SOLID principles where applicable.

### 18. System Lifecycle Awareness
- Understand full flow: Request -> Router -> Controller -> Service -> Repository -> DB.
- Know cache layer behavior (Redis/Memcached) — when and what to cache, invalidation strategy.
- Background jobs & queues for heavy work (BullMQ, Celery).
- Be able to identify where latency and failures happen in the flow.

### 19. AI-Generated Code Verification
- Don't blindly trust AI output — treat as second opinion.
- Verify against official docs before shipping.
- Compare with project's existing architecture and patterns.
- Ask: What alternatives exist? Does this match industry standards?

### 20. Legacy System Safety
- Understand existing data behavior before changing anything.
- Protect backward compatibility — old consumers still depend on it.
- Use versioning, migration plans, feature flags for safe rollouts.

---

## Output Format

For each issue found:
1. **Rule violated** — which rule from above
2. **File & line** — exact location
3. **Why it matters** — real-world consequence (security breach, data leak, performance crash, etc.)
4. **Bad code** — the current problematic code
5. **Fixed code** — the corrected version

End with a summary: total issues found, severity breakdown (critical/warning/info), and top 3 priorities to fix first.
