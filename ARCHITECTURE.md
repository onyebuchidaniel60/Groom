# GROOM — ARCHITECTURE & TECHNICAL DESIGN

**Status:** Proposed fixed architecture v1.0. Changes require operator approval.  
**Principle:** A modular monolith is sufficient. Do not introduce microservices, Redis, a separate API server or an LLM.

---

## 1. Architecture overview

### 1.1 Chosen architecture

- Next.js App Router application is the web frontend and server-side API boundary.
- Supabase provides managed PostgreSQL, Auth and private object storage.
- Inngest runs durable, retriable background work for YouCam upload, task creation, polling, result storage and cleanup.
- Perfect Corp YouCam provides the two required virtual grooming transformations.
- Vercel hosts the application and preview/production deployments.

```text
Owner browser                     Participant browser
     |                                    |
     +-------------- HTTPS ---------------+
                      |
             Next.js (Vercel)
       pages + route handlers + domain layer
          |             |             |
          |             |             +---- Inngest event endpoint
          |             |                         |
          |             |                   Inngest worker
          |             |                         |
          |             |                    YouCam REST
          |             |                 file upload + VTO task
          |             |                         |
          +-------------+-------------------------+
                        |
                 Supabase services
            PostgreSQL + Auth + Storage
```

### 1.2 Trust boundaries

1. Browser is untrusted. It never receives YouCam API keys, service-role credentials, raw DB access or private storage paths.
2. Next.js route handlers authenticate owner or participant session and apply Zod validation and domain permissions.
3. Supabase service-role credentials are server-only and used only in narrowly scoped server modules. Prefer user-scoped Supabase clients for owner reads/writes so RLS remains active.
4. Inngest workers are trusted server processes but validate event payloads and reload authoritative DB state before work.
5. YouCam is an external processor. Send only the participant photo and required template/task parameters; do not send names, emails, experiment business metadata or votes.
6. Supabase private buckets contain transient source/result images. No public buckets.

---

## 2. Fixed technology stack

| Layer | Technology | Responsibility / decision |
|---|---|---|
| Runtime | Node.js 22 LTS | Stable server runtime; matches current Next.js requirements. |
| Framework | Next.js 16.4.x App Router | One deployable app for frontend and route handlers; no separate backend service. Pin exact compatible patch at Phase 1 and commit lockfile. |
| UI | React 19.3-compatible version | Component rendering; use stable release resolved by Next.js peer dependencies. |
| Language | TypeScript strict | Shared input/output types; no implicit any. |
| Styling | Tailwind CSS 4.x | Temporary functional styling initially; final design only after mockup approval. |
| Validation | Zod | Parse all request bodies, params, env and external provider responses. |
| Database | Supabase PostgreSQL | Relational state, transactional writes, indexes, RLS. |
| Auth | Supabase Auth email/password | Owner authentication; no participant accounts. |
| SSR Auth integration | `@supabase/ssr` behind `src/lib/supabase/` adapters | Server/browser clients. Documentation currently labels the package beta; isolate it and pin tested version. |
| Object storage | Supabase Storage | Private `respondent-inputs` and `respondent-renders` buckets. |
| Durable jobs | Inngest TypeScript SDK v4 | Background processing, retry, sleep/polling, concurrency and cleanup. |
| Image processing | `sharp` | Decode/re-encode JPEG, resize and strip EXIF. |
| Unit/integration tests | Vitest | Domain rules, route handlers, provider adapters and DB integration tests. |
| Component tests | React Testing Library | Interaction and state tests. |
| E2E | Playwright | Owner and participant workflows against preview/test deployment. |
| Hosting | Vercel | Stable Preview and Production environments. |
| Source control | GitHub | Repository and CI workflow. |

**Rejected alternatives:** Separate Express/Fastify API (unnecessary second deployment); Firebase (Postgres transactions and RLS better fit relational ownership/aggregate reporting); Redis/queue (Inngest is sufficient); Stripe (no payments); OpenAI/LLM (no generative reasoning requirement); custom ML (YouCam performs the image transformations).

**Version discipline:** Do not install `latest` packages in implementation tasks. Pin exact versions in `package.json` and lockfile after a clean install/build. Next.js 16 requires Node 20.9+; Node 22 LTS is the chosen runtime. Verify versions against current release notes during Phase 0/1.

---

## 3. Database model

All tables use UUID primary keys unless stated. PostgreSQL timestamps use `timestamptz` UTC. `created_at` defaults to `now()`; mutable tables have `updated_at` maintained by DB trigger. Enable RLS on every application table. Define migrations in `supabase/migrations/`; do not edit production schema manually.

### 3.1 `profiles`

- `id uuid PK references auth.users(id) on delete cascade`
- `business_name text not null` (2–100 chars)
- `business_type text not null check in ('brand','barbershop','other')`
- `website_url text null`
- `created_at`, `updated_at timestamptz not null`
- RLS: owner can select/update own row only; insert requires `id = auth.uid()`.

### 3.2 `style_templates`

Curated catalogue of real templates fetched from YouCam API and reviewed by operator. Do not assume the provider template response has stable marketing labels.

- `id uuid PK`
- `experiment_type text not null check in ('hairstyle','beard')`
- `provider text not null default 'perfectcorp'`
- `provider_template_id text not null`
- `display_name text not null`
- `description text null`
- `preview_asset_path text null` (private or public-safe preview asset only if rights allow)
- `men_verified boolean not null default false`
- `active boolean not null default false`
- `metadata jsonb not null default '{}'` (sanitised fields only; never API credentials)
- `created_at`, `updated_at`
- Unique `(provider, experiment_type, provider_template_id)`.
- Index `(experiment_type, active, men_verified)`.
- Owner can read active verified rows; only service role/operator can insert/update.

### 3.3 `experiments`

- `id uuid PK`
- `owner_id uuid not null references profiles(id)`
- `title text not null`
- `description text null`
- `participant_instructions text null`
- `experiment_type text not null check in ('hairstyle','beard')`
- `status text not null check in ('draft','published','paused','closed','archived') default 'draft'`
- `public_slug text unique null` (at least 128 bits of random entropy; use 18 random bytes base64url)
- `response_target int not null check between 10 and 50 default 20`
- `opens_at timestamptz null`, `closes_at timestamptz null`
- `published_at timestamptz null`, `closed_at timestamptz null`, `archived_at timestamptz null`
- `active_session_count int not null default 0 check >=0`
- `completed_response_count int not null default 0 check >=0`
- `created_at`, `updated_at`
- Index `(owner_id, status, created_at desc)` and partial unique/index on `public_slug`.
- RLS: owner can select/update own experiments. Public participant reads go only through server route handler using explicitly selected safe columns. Owner cannot alter `owner_id`, status via generic PATCH, counters or slug.

### 3.4 `experiment_options`

- `id uuid PK`
- `experiment_id uuid not null references experiments(id) on delete cascade`
- `template_id uuid not null references style_templates(id)`
- `display_label text not null` (2–60 chars; owner-entered alias allowed but must not misrepresent provider output)
- `display_position smallint not null check 1..4`
- `created_at`
- Unique `(experiment_id, template_id)` and `(experiment_id, display_position)`.
- Index `(experiment_id, display_position)`.
- Option rows immutable after experiment is published.

### 3.5 `respondent_sessions`

- `id uuid PK`
- `experiment_id uuid not null references experiments(id)`
- `token_hash text not null unique` (HMAC-SHA256 of opaque token; never store raw token)
- `creation_request_id uuid not null` (client idempotency UUID, unique with experiment ID)
- `status text not null check in ('created','consented','uploaded','rendering','ready','submitted','failed','expired')`
- `consent_version text not null`
- `consented_at timestamptz not null`
- `age_confirmed boolean not null check (age_confirmed)`
- `hair_texture_self_report text null check in ('straight','wavy','curly','coily','unsure','prefer_not_to_say')`
- `input_object_path text null`
- `upload_token_hash text null` (HMAC hash of one-time upload completion token)
- `upload_token_expires_at timestamptz null`
- `upload_count smallint not null default 0 check between 0 and 2`
- `source_file_size_bytes int null`
- `display_order uuid[] not null` (all option IDs exactly once)
- `expires_at timestamptz not null` (created_at + 2h)
- `failure_code text null` (safe internal enum; no raw provider error)
- `created_at`, `updated_at`, `submitted_at timestamptz null`
- Index `(experiment_id, status, created_at)` and `(status, expires_at)`.
- Owner cannot select individual sessions. Participant routes resolve the session via token hash using server-side service access; no public table grants.

### 3.6 `ai_jobs`

One job per participant-session-option.

- `id uuid PK`
- `session_id uuid not null references respondent_sessions(id) on delete cascade`
- `option_id uuid not null references experiment_options(id)`
- `provider text not null default 'perfectcorp'`
- `endpoint_key text not null check in ('hairstyle_v2_1','beard_v1_0')`
- `provider_file_id text null`
- `provider_task_id text null unique`
- `status text not null check in ('queued','unit_reserved','file_uploaded','task_created','running','succeeded','failed','unknown_create_outcome')`
- `reserved_units int not null check >=0`
- `result_object_path text null`
- `poll_count int not null default 0`
- `last_polled_at timestamptz null`
- `safe_error_code text null`, `safe_error_message text null`
- `created_at`, `updated_at`, `completed_at timestamptz null`
- Unique `(session_id, option_id)`.
- Index `(status, created_at)` and `(session_id)`.

### 3.7 `responses`

- `id uuid PK`
- `session_id uuid not null unique references respondent_sessions(id)`
- `experiment_id uuid not null references experiments(id)`
- `first_choice_option_id uuid not null references experiment_options(id)`
- `submitted_at timestamptz not null default now()`
- Check consistency of first-choice option to same experiment in transaction/trigger.
- Owner has no row-level select grant. Aggregate SQL function `get_experiment_aggregate_results(experiment_id)` uses `SECURITY DEFINER` only with fixed `search_path`, checks `auth.uid()` against the experiment owner inside the function, returns grouped values only, and is executable by `authenticated` role only. No raw response rows are exposed.

### 3.8 `response_ratings`

- `id uuid PK`
- `response_id uuid not null references responses(id) on delete cascade`
- `option_id uuid not null references experiment_options(id)`
- `rating smallint not null check 1 <= rating <= 5`
- Unique `(response_id, option_id)`.
- No owner direct select; aggregate-only function.

### 3.9 `usage_ledger`

- `id uuid PK`
- `ai_job_id uuid not null unique references ai_jobs(id)`
- `units int not null check >0`
- `status text not null check in ('reserved','committed','released','unknown')`
- `reservation_key text not null unique`
- `provider_task_id text null`
- `created_at`, `updated_at`, `settled_at timestamptz null`
- Service role only. No client access.

### 3.10 `api_usage_budget`

Single row keyed `provider + environment`.
- `provider text`, `environment text`, composite PK.
- `budget_units int not null`, `reserved_units int not null default 0`, `committed_units int not null default 0`, `updated_at`.
- Atomic conditional update ensures `reserved + committed + request <= budget`.
- Production and preview budgets are separate; preview uses a very small explicit test budget and ideally a separate API key/account if the provider permits.

### 3.11 `rate_limit_buckets`

Short-lived application rate-limit counters. Store only an HMAC of the source IP/session key, not the raw IP.

- `id uuid PK`
- `scope text not null` (e.g. `public_session_create`, `public_page_read`)
- `key_hash text not null` (HMAC-SHA256 of client IP or participant/session fingerprint using `RATE_LIMIT_HMAC_SECRET`)
- `window_start timestamptz not null`
- `request_count int not null default 0`
- `expires_at timestamptz not null`
- Unique `(scope, key_hash, window_start)`; index on `expires_at`.
- Service role only; scheduled cleanup deletes expired rows daily. The HMAC value is still pseudonymous operational data; minimise retention.

### 3.12 `audit_events`

- `id uuid PK`
- `actor_user_id uuid null` (owner or operator)
- `experiment_id uuid null`
- `event_type text not null`
- `metadata jsonb not null default '{}'` (allowlisted IDs/statuses only; no images, tokens, email, IP or raw request payloads)
- `created_at timestamptz not null default now()`
- Insert-only; service role only.

### 3.13 Relationships

```text
auth.users 1──1 profiles 1──N experiments 1──N experiment_options N──1 style_templates
                                      |
                                      1──N respondent_sessions 1──N ai_jobs
                                                   |
                                                   1──0..1 responses 1──N response_ratings

ai_jobs 1──1 usage_ledger
all lifecycle actions -> audit_events; rate-limits -> rate_limit_buckets
```

### 3.14 Data classification

| Data | Classification | Who may access |
|---|---|---|
| Owner email/auth identity | Private personal data | Supabase Auth; owner sees own account only; operator only for support/security. |
| Business name/website | Owner-controlled business data | Public-safe business display name may appear on published experiment; email never exposed. |
| Participant selfie | Sensitive personal image data | Participant during upload/preview flow; server/YouCam processor only as necessary; not experiment owner. |
| Generated renders | Private personal image data | Participant only until submit/expiry; never owner. |
| Consent version/time and age affirmation | Private compliance record | Server/operator; owner does not see respondent-level record. |
| Ratings and first choice | Research response data | Owner only as aggregated counts/means; individual row never exposed. |
| Session token | Secret credential | Browser cookie and server hash only. |
| Provider keys/task credentials | Secret | Server environment and worker only. |
| Usage/audit logs | Operational | Operator only; redacted. |

---

## 4. API design

All Groom routes use JSON except the signed storage upload. All JSON errors follow:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please correct the highlighted fields.",
    "requestId": "server-generated-id",
    "details": [{"field": "title", "issue": "Required"}]
  }
}
```

Do not return stack traces, provider raw errors, SQL, storage paths or secrets. Use HTTP status codes consistently.

### 4.1 Owner APIs

| Method/path | Purpose | Auth / permission | Main validation and effects |
|---|---|---|---|
| `GET /api/style-templates?type=hairstyle\|beard` | Active verified template list | Owner session | Returns curated public-safe catalogue fields, never raw provider credentials. |
| `GET /api/experiments` | List owner's experiments | Owner session | Owner filter enforced server-side and RLS. |
| `POST /api/experiments` | Create draft | Owner session | Zod body; transaction inserts experiment/options; 3–4 distinct active templates of same type. |
| `GET /api/experiments/:id` | Owner experiment detail | Owner owns resource | 404 for foreign ID to reduce enumeration. |
| `PATCH /api/experiments/:id` | Edit draft text/target/options | Owner owns draft | Published options/category immutable; counters/status/server fields forbidden. |
| `POST /api/experiments/:id/publish` | Publish draft | Owner owns resource | Lock and revalidate; random slug; audit. |
| `POST /api/experiments/:id/pause` | Pause new starts | Owner owns resource | Only published -> paused. |
| `POST /api/experiments/:id/resume` | Resume | Owner owns resource | Only paused -> published when not target/date closed. |
| `POST /api/experiments/:id/close` | Close new starts | Owner owns resource | Published/paused -> closed; terminal. |
| `POST /api/experiments/:id/archive` | Archive closed experiment | Owner owns resource | Only closed -> archived. |
| `GET /api/experiments/:id/results` | Aggregate results | Owner owns resource | Suppression thresholds; aggregate only. |
| `GET /api/health` | Health check | Public, no internals | Returns `{status:'ok'}` and version; no env values. |

`POST /api/experiments` body:

```json
{
  "title": "New signature fade collection",
  "description": "Help us choose which new styles to introduce.",
  "participantInstructions": "Compare each preview and select the one you prefer.",
  "experimentType": "hairstyle",
  "responseTarget": 20,
  "options": [
    {"templateId": "uuid", "displayLabel": "Style A"},
    {"templateId": "uuid", "displayLabel": "Style B"},
    {"templateId": "uuid", "displayLabel": "Style C"}
  ]
}
```

### 4.2 Participant APIs

| Method/path | Purpose | Auth / permission | Main validation and effects |
|---|---|---|---|
| `GET /api/public/experiments/:slug` | Public-safe experiment info | Public | Only published and accepting studies; never expose owner personal data or provider IDs. |
| `POST /api/public/experiments/:slug/sessions` | Record consent and create anonymous session | Public + rate limit | Requires `consentAccepted=true`, `ageConfirmed=true`; optional allowed hair texture. Sets secure participant cookie. |
| `POST /api/participant/upload-url` | Create private signed upload destination | Participant cookie | Generates server-owned random object path; checks session state/expiry; file size/type declaration is only preliminary. |
| `POST /api/participant/upload-complete` | Finalize upload and enqueue render jobs | Participant cookie | Server verifies object exists, downloads and inspects bytes, normalises; creates jobs idempotently and emits Inngest event. |
| `GET /api/participant/session` | Retrieve own session state and temporary render URLs | Participant cookie | Signed URLs short TTL; no arbitrary session ID accepted without cookie-bound hash match. |
| `POST /api/participant/submit` | Submit ratings + first choice | Participant cookie | Idempotency key; atomic transaction; one response only. |

`POST /api/public/experiments/:slug/sessions` body:

```json
{
  "consentAccepted": true,
  "ageConfirmed": true,
  "hairTexture": "wavy"
}
```

`POST /api/participant/upload-url` body:

```json
{"contentType":"image/jpeg","fileSize":2345678}
```

Response contains a time-limited signed upload URL and opaque upload token. It does not disclose the bucket's service key or reusable storage credentials.

`POST /api/participant/upload-complete` body:

```json
{"uploadToken":"opaque-one-time-token"}
```

`POST /api/participant/submit` body:

```json
{
  "idempotencyKey": "random-client-generated-uuid",
  "firstChoiceOptionId": "uuid",
  "ratings": [
    {"optionId":"uuid","rating":5},
    {"optionId":"uuid","rating":3},
    {"optionId":"uuid","rating":4}
  ]
}
```

### 4.3 Inngest endpoint and events

- `GET/POST/PUT /api/inngest` as required by the installed Inngest Next.js adapter.
- Event `groom/session.render_requested`: `{sessionId}` only. Worker reloads source image, options and consent status from database; never trusts event-supplied owner/template values.
- Event `groom/session.cleanup_requested`: `{sessionId}`.
- Scheduled event `groom/cleanup.expired`: no participant data; selects expiry candidates server-side.

### 4.4 Rate limits

- Owner sign-in/signup: provider protections plus application IP/account throttling; never expose whether an email exists.
- Public experiment GET: 60/minute/IP as default edge/application limit.
- Session creation: 5/hour/IP/slug using a keyed HMAC rate-limit bucket, plus one active participant session per browser at a time. Cookie name is `groom_participant`, path `/api`, max-age 7200 seconds. After submission/expiry the cookie is cleared; a new session may then be started. Cookie deletion/incognito can bypass person-level uniqueness.
- Upload URL and upload-complete: 5/session/minute.
- Submit: 5/session/minute with idempotency; actual successful response one only.
- Image generation: global Inngest concurrency 2 until provider-specific per-account limit verified; do not exceed documented provider rate limits.
- Do not persist raw IP addresses. Use HMAC-derived short-lived keys in `rate_limit_buckets`, expire/delete them daily and treat them as pseudonymous operational data.

### 4.5 Idempotency

- `upload-complete` uses unique upload token and session state; repeated call returns current state.
- Session creation uses client `idempotencyKey` idempotency key scoped to slug and short time window.
- `submit` uses unique session response constraint plus idempotency key.
- Inngest job per `(session_id, option_id)` unique; retries resume existing job state.
- Provider task creation has no assumed idempotency guarantee. Persist state before and after POST; ambiguous response leads to `unknown_create_outcome`, never automatic duplicate POST.

---

## 5. Database operations, transactions and races

### 5.1 Create experiment

1. Validate owner session and body.
2. Begin transaction.
3. Select selected template IDs `FOR SHARE`; verify active, men's verified, correct category.
4. Insert experiment draft.
5. Insert 3–4 options with unique positions.
6. Commit. On any error rollback all rows.

### 5.2 Publish

1. Begin transaction and lock experiment row `FOR UPDATE`.
2. Verify owner/status/options/count/template state.
3. Enforce owner active-publication limit under lock/unique transactional strategy.
4. Generate slug with cryptographic RNG; retry on unique collision.
5. Set status and timestamps; insert audit event.
6. Commit.

### 5.3 Create participant session / reserve target slot

1. Resolve public slug and lock experiment row.
2. Verify published, date open and budget policy permits.
3. If the client supplies a valid `groom_participant` cookie, resolve its HMAC. If it maps to an active session, reject a new session; if terminal/expired, clear it. A repeated `idempotencyKey` for the same experiment returns the existing session and re-sets its cookie.
4. Under row lock verify `completed_response_count + active_session_count < response_target`.
5. Insert session with token hash, unique `creation_request_id`, expiry and random display order.
6. Increment `active_session_count` atomically.
7. Commit; set cookie only after successful commit.

When a session expires/fails or successfully submits, decrement active count exactly once using guarded state transition. Use transaction and `UPDATE ... WHERE status IN (...)`; only the request that changes the row may adjust counters.

### 5.4 Reserve unit budget

1. Lock budget row or use atomic conditional update.
2. Check `budget_units - reserved_units - committed_units >= required_units`.
3. Increment reserved units; insert unique usage ledger row and mark job unit_reserved in same transaction.
4. If insufficient, set safe failure `API_BUDGET_EXHAUSTED` without calling YouCam.
5. On verified successful billing, move reserved to committed. On verified non-billable failure, release. Unknown outcome remains reserved/unknown for reconciliation.

### 5.5 Submit response

1. Validate participant cookie → token hash → unexpired ready session.
2. Validate every option belongs to the same experiment; all ratings 1–5; exactly one first choice.
3. Begin transaction; lock experiment and session.
4. Recheck not submitted, target slot valid, session ready and experiment state permits existing session completion.
5. Insert response (unique session ID).
6. Insert rating rows.
7. Mark session submitted; update completed count and active count exactly once.
8. If completed count reaches target, transition published/paused experiment to closed.
9. Insert audit event, commit.
10. Enqueue cleanup after commit.

If client retries after network timeout, query existing response for session and return success; do not insert a second response.

### 5.6 Concurrency and isolation

- Use PostgreSQL transactions and row locks for publish, response-slot reservation, submission, counters and usage budget.
- Do not calculate counters by read-then-write in application code without lock or atomic update.
- `unique` constraints are the final backstop against duplicate sessions/responses/jobs.
- Aggregate results query committed response rows only.
- If Inngest emits duplicate events, worker resumes persisted job; it does not create another job or provider task unless it can prove no task was created.

---

## 6. Authentication and authorization

### Owner
- Supabase Auth email/password; email verification setting enabled before public launch if configuration allows.
- Server-side Supabase SSR session cookie with HttpOnly/Secure/SameSite defaults; do not trust client-supplied owner ID.
- Every owner query includes `owner_id = auth.uid()` and RLS enforces same.
- Resource access to foreign experiment returns 404 rather than revealing existence.
- Generic PATCH cannot modify owner ID, status, slug, counters, timestamps or results.

### Participant
- No account, name, email or password. Groom cannot prove one participant equals one real-world person; cookie deletion/incognito may allow another session. This is a disclosed anti-fraud limitation.
- After affirmative consent and 18+ self-attestation, server creates a random session UUID and derives an opaque 256-bit token with HMAC-SHA256 over session UUID + experiment UUID + client `idempotencyKey`, keyed by `PARTICIPANT_TOKEN_HMAC_SECRET`. Store only a purpose-separated HMAC hash of the token. This makes an idempotent retry capable of re-deriving the same token without storing plaintext. Browser receives the URL-safe token in an HttpOnly cookie named `groom_participant`, with `Path=/api`, max-age 7200 seconds, `Secure` in production and `SameSite=Lax`.
- - Participant APIs require cookie and compare server-computed hash in constant time where applicable. Cookie is named `groom_participant`, scoped to `Path=/api`, and cleared on submission/expiry. Only one active session per browser at a time; cookie deletion/incognito can bypass one-human-one-response enforcement.
- Token rotation on session restart is not allowed; one session is one response.
- Cookie theft is mitigated by TLS, HttpOnly and short TTL; participant can see only that session's temporary renders.

### Operator
- Uses Supabase dashboard/Vercel/Inngest/secret manager with MFA.
- No default user-facing admin role or admin UI. Do not seed an `admin` boolean in profiles as a shortcut.
- Catalogue maintenance is a controlled migration/seed or protected operator script, audited.

---

## 7. Security architecture and threat model

| Threat | Mitigation / required test |
|---|---|
| IDOR / another owner's experiment | Owner ID derived from Auth; RLS; foreign IDs return 404; test cross-account reads and mutations. |
| Participant session theft/replay | Opaque 256-bit token, HMAC hash at rest, HttpOnly/Secure/SameSite cookie, 2h TTL, scoped path, one-response DB uniqueness. |
| SQL injection | Supabase parameterised query APIs; no string-built SQL; validate all inputs; review any RPC SQL. |
| XSS | React escaping; avoid unsafe HTML; sanitise/limit user text; CSP; render provider strings as text only. |
| CSRF | SameSite=Lax, origin check for mutating requests, CSRF token/header for owner APIs as required; never rely on CORS alone. |
| Brute force / enumeration | Supabase protections, rate limits, uniform auth errors, random public slugs, no sequential public IDs. |
| Secret exposure | Server-only env; never prefix secret with `NEXT_PUBLIC_`; scan bundles and git diff; separate Preview/Production keys. |
| Malicious image / upload bomb | Enforce byte cap, signature sniffing, `sharp` decode/re-encode in isolated server path, dimensions/pixel cap, reject SVG/GIF/HEIC, timeouts. |
| SSRF | Do not accept arbitrary URLs. Provider output download only from exact approved hostname(s) and HTTPS; block redirects to private/link-local IPs; resolve and revalidate DNS/redirect targets. |
| Provider task replay / duplicate billing | Persist job state and provider task ID; no automatic repeated POST on ambiguous outcome; unique job constraint. |
| API budget abuse | Reserve unit ledger transactionally, global cap, per IP/session rate limit, concurrency cap. |
| Data disclosure | Private buckets, short-lived signed URLs, owner aggregate-only reporting, no participant-level endpoint. |
| Prompt injection | Not applicable: no LLM, no user-authored prompt or external agent. Do not add LLM to MVP. |
| Privilege escalation | No role field accepted from client; operator access outside product; RLS and server checks. |
| Race conditions | Row locks, unique constraints, atomic counters and budget updates; concurrency tests. |
| Dependency compromise | Lockfile, Dependabot/GitHub security alerts where available, minimal dependencies, review package scripts. |
| Log/telemetry leakage | Redact cookies, tokens, image URLs, signed upload URLs, API keys, images, emails and raw provider responses. |
| Abuse / harmful content | Restrict to self-image upload and grooming preview; no public image gallery or image sharing. Provide report/support route and abuse response procedure. |

### Privacy requirements
- Treat source photos and generated face renders as personal data. Groom does not use them to identify a person.
- Explicit, unchecked consent before collection; say that image is sent to Perfect Corp for hairstyle/beard simulation; explain purpose, retention, withdrawal path and that owner receives aggregate findings only.
- Separate 18+ affirmation; do not collect government ID.
- Optional hair texture self-report is not required for participation and includes “prefer not to say.”
- Do not infer race, ethnicity, gender identity or age from image; the product is marketed for adult men's grooming, but does not infer sex/gender from the face.
- Store no raw IP. No tracking pixels or third-party marketing analytics in MVP.
- Data minimisation; image deleted from Groom after generation is complete and no longer needed, or at session expiry. Result previews deleted on submit/expiry.
- Provide a privacy contact and withdrawal/delete request mechanism. Because participant contact is not collected, a deletion request must include the participant session reference held by the user; document the operational limitation.
- Verify Perfect Corp processor terms, hosting regions, retention, deletion, permitted use and no-training/secondary-use terms before public collection. Do not make claims not supported by the provider contract.
- Nigeria Data Protection Act 2023 applies to Nigerian processing; obtain qualified legal review before commercial launch or expansion to other jurisdictions. This blueprint is an engineering design, not legal advice.

---

## 8. YouCam integration architecture

### 8.1 Provider operations

| Capability | Endpoint | Units (current docs) | Constraints |
|---|---|---:|---|
| Hairstyle preset V2.1 | `POST /s2s/v2.1/task/hair-transfer`; templates `GET /s2s/v2.1/task/template/hair-transfer`; status `GET /s2s/v2.1/task/hair-transfer/{task_id}` | 2 per preset task | JPG/JPEG, <10MB, long side <=1024, one full face; face width >=128; pose bounds per docs; shoulders and hairstyle visible. |
| Beard style V1.0 | `POST /s2s/v2.0/task/beard-style`; templates `GET /s2s/v2.0/task/template/beard-style`; status `GET /s2s/v2.0/task/beard-style/{task_id}` | 2 per task | JPG/JPEG, <10MB, long side <1024, face width >256, one full face, yaw within ±30. Poll inside provider retention period. |
| File upload | `POST /s2s/v2.0/file`, then HTTP PUT bytes to returned signed `requests.url` | No separate unit assumed; verify account terms | Calling File API alone does NOT upload bytes. Confirm PUT success before task POST. |

Exact request properties must be generated from current official OpenAPI/docs and live API POC; do not invent fields. Provider adapter normalises differences between endpoint versions.

### 8.2 Provider adapter interface

`src/server/integrations/youcam/` exposes:
- `listTemplates(type): Promise<ProviderTemplate[]>`
- `uploadSourceImage(bytes, mimeType): Promise<ProviderFile>`
- `createHairstyleTask(fileId, providerTemplateId): Promise<ProviderTaskRef>`
- `createBeardTask(fileId, providerTemplateId): Promise<ProviderTaskRef>`
- `getTaskStatus(type, taskId): Promise<ProviderTaskStatus>`
- `downloadResult(approvedProviderUrl): Promise<Buffer>`

Zod-validate every provider response. Persist only required IDs/status. Store sanitized provider errors as error enum + limited message. Never persist signed result URLs beyond the current worker step.

### 8.3 Upload pipeline

1. User uploads source to private Supabase bucket using signed upload URL generated by Groom.
2. `upload-complete` worker downloads object through service credentials.
3. `sharp` decodes image, enforces limits, strips EXIF and re-encodes JPEG.
4. Call YouCam File API to obtain provider `file_id` and signed PUT URL.
5. PUT actual JPEG bytes to `requests.url`; check successful status.
6. Only then call task endpoint with `src_file_id` and selected provider `template_id`.
7. Store task ID, poll according to provider-required interval/retention.
8. On success, fetch result URL immediately from exact allowlisted host, download with byte/time limit, store in private Supabase bucket and delete local provider URL from memory/logs.
9. Delete participant source object when every job is terminal and source is no longer needed.

### 8.4 Polling and retry strategy

- Inngest durable function polls with 5–10 second intervals initially; validate actual provider guidance/retention and tune in Phase 0.
- Global concurrency is 2 until account rate limits are confirmed.
- Max task wall time and poll count are configurable and recorded; stop before provider retention deadline.
- Retry GET status, temporary network error and result download with bounded exponential backoff and jitter.
- Do not retry validation errors or unsupported image failures.
- If task POST request times out before task ID is received, set `unknown_create_outcome`; do not POST again automatically.
- If provider task returns terminal error, mark failed and use documented unit billing semantics to reconcile reserved units.
- If one of 3–4 option renders fails terminally, mark the whole participant session failed, do not show partial comparison. A manual/controlled recovery can restart the full set only if the operator confirms no outstanding task and enough units remain.

### 8.5 Template catalogue verification

The official template APIs return current template identifiers. The project must call them using the real API key, inspect actual metadata and sample outputs, and curate a static database catalogue of verified men's options. Do not assume a field such as `gender=male` exists. Operator manually marks each chosen template `men_verified=true` after visual QA. Disable any template lacking an approved name/preview/license check. No placeholder or guessed provider IDs.

---

## 9. Frontend / UX architecture

### 9.1 Routes

| Route | Access | Purpose |
|---|---|---|
| `/` | Public | Product explanation and owner CTA. |
| `/sign-up` | Public | Owner registration. |
| `/sign-in` | Public | Owner login. |
| `/dashboard` | Owner | Experiments list, status, create CTA. |
| `/dashboard/experiments/new` | Owner | Draft creation. |
| `/dashboard/experiments/[id]` | Owner | Detail, share link, lifecycle actions. |
| `/dashboard/experiments/[id]/results` | Owner | Aggregate results and suppression state. |
| `/p/[slug]` | Public/participant | Experiment intro and participant flow. |
| `/docs` | Public | Product/participant and operator documentation index. |
| `/privacy` | Public | Privacy notice. |
| `/terms` | Public | Terms and appropriate disclaimers. |
| `/support` | Public | Support contact/instructions. |
| `not-found` | Public | Branded not-found page. |

Participant sub-states may be route segments or client state on `/p/[slug]`; keep the public URL shareable and avoid exposing session IDs in the URL.

### 9.2 Screen requirements

**Home:** Clear product statement; explain virtual preference research and limitations; CTA for business owner; sign-in; footer; no fabricated testimonials/logos/results.

**Sign-up/sign-in:** Email/password fields, password visibility toggle, loading, validation and generic auth error; keyboard accessible.

**Dashboard:** Experiment list; state badges; response count / target; empty state; create CTA; no individual participant data.

**Create experiment:** Step structure: details → category → choose 3–4 templates → target → review/save draft. Changing category clears incompatible option selection with warning. Error and loading states for template fetch.

**Experiment detail:** Status, category, options, response target, participant link, copy action, lifecycle controls. Confirm irreversible close/archive. No option editing post-publish.

**Results:** Overall results locked under 5 responses; progress count; per-option average and distribution; first-choice share; segment charts only at N≥10. Explain results as directional preference, not statistical market proof. CSV only if built and only aggregate.

**Participant intro:** Business name, experiment title and instructions; “18+ only”; privacy/terms links; start button; unavailable/paused/closed/target reached states.

**Consent:** Clear affirmative, unchecked consent; plain language says photo is processed by Groom and Perfect Corp for virtual style rendering and deleted from Groom when no longer needed; age checkbox separate. Consent version stored.

**Photo capture/upload:** Short image instructions, JPG requirement, file size, preview before upload, remove/replace. Permission denied and invalid file help; no camera permission required because file selection is sufficient MVP.

**Processing:** Progress status per option with friendly language; do not expose provider IDs; retry only if server authorises; explain it may take a short time and offer refresh. Full failure state provides safe restart/support path.

**Compare and vote:** All 3–4 previews, random stable order, each with 1–5 rating control; exactly one “my favourite” radio; submit disabled until complete; explicit “AI preview may differ from a real barber result” disclaimer.

**Thank you:** Confirmation and deletion statement; no share result or individual public gallery.

**Docs/privacy/terms/support:** Real pages, not placeholder routes. Terms include age restriction, permitted use, image processing disclosure, limitations and contact.

### 9.3 Shared behavior components

- `AppHeader`, `OwnerNav`, `AppFooter`, `PublicFooter`.
- `AuthForm`, `FormField`, `FormError`, `PasswordField`.
- `ExperimentStatusBadge`, `ExperimentCard`, `ExperimentForm`, `CategorySelect`.
- `TemplatePicker`, `TemplateOptionCard`, `ResponseTargetInput`.
- `ShareLinkPanel`, `ConfirmActionDialog`, `CopyButton`.
- `ConsentPanel`, `AgeGate`, `ImageUpload`, `ImageGuidance`, `UploadProgress`.
- `RenderProgressList`, `PreviewComparisonGrid`, `RatingInput`, `FavouriteRadio`.
- `ResultsGate`, `AggregateMetricCard`, `RatingDistribution`, `FirstChoiceChart`, `SuppressedSegmentNotice`.
- `LoadingState`, `EmptyState`, `ErrorState`, `NotFoundState`, `Toast/InlineAlert`.

### 9.4 UI states

Every route must implement relevant loading, empty, validation, success, error, permission-denied, unavailable, not-found and expired states. No inert controls. While functionality is incomplete in early preview, placeholders must be visibly labelled “Not connected yet” and must not imply a successful API action.

### 9.5 Design sequencing

- Phase 1 skeletal UI uses restrained temporary neutral design tokens only.
- Core behavior implemented and E2E tested before final visual design.
- Collect operator references after functionality gate.
- Produce mockups for home, dashboard, experiment builder, public intro/consent, upload/processing, compare/vote and results, including desktop and mobile.
- Wait for explicit written operator approval.
- Final styling cannot alter API, data model, workflow or permissions.

### 9.6 Footer/link requirements

- `/docs`, `/privacy`, `/terms`, `/support` are real routes.
- Footer includes Documentation → `/docs`.
- GitHub link appears only when an actual intended public repository URL is configured. Never invent it; never expose private repo without approval. If repo is private, omit public link with an explicit recorded reason and offer a contact/support route instead, subject to operator approval.
- Verify all links in Playwright and deployed preview.

---

## 10. Third-party integrations and deployment

### Required

**Supabase**
- Data exchanged: Auth credentials/session metadata, experiment records, session state and encrypted/private image objects.
- Env: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY` (client safe by design), `SUPABASE_SERVICE_ROLE_KEY` server-only.
- RLS mandatory; private buckets; signed upload/download URLs with short TTL.
- Failure: fail closed for write/owner access; do not expose service key; clear retry guidance.

**Perfect Corp YouCam**
- Auth: Bearer API key server-side.
- Env: `YOUCAM_API_BASE_URL`, `YOUCAM_API_KEY` secret, `YOUCAM_UNIT_BUDGET`, task cost constants.
- Data: processed selfie bytes, provider template ID, task status/result. No owner email or survey data.
- Failure: durable jobs, explicit error state; no duplicate task POST after ambiguous outcome.

**Inngest**
- Env: `INNGEST_EVENT_KEY`, `INNGEST_SIGNING_KEY` server-only.
- Purpose: render orchestration, polling and cleanup.
- Verify signatures using official SDK; do not accept unsigned events.

**Vercel**
- Environment split: Preview vs Production, exact deployments/branches. Preview uses safe test keys/data and protection where available. No production API budget or production database in Preview.

### No MVP integration

Stripe, email marketing, WhatsApp, SMS, social login, OpenAI, Google Analytics, Meta Pixel, Sentry (optional later), external search or public image hosting.

### Preview deployment strategy

- Use GitHub integration to Vercel.
- First deploy after the route-complete shell is navigable, even before backend is finished.
- Record exact URL, commit SHA and visible/incomplete routes in AI_HANDOFF.md.
- Protect Vercel Preview with Vercel Deployment Protection if available.
- Use separate Supabase project for preview or strict isolated preview schema/bucket, plus separate low-limit YouCam test key/budget if permitted. Never point Preview at production participant records.
- Update preview after each verified phase; no promotion to Production without explicit operator instruction.

---

## 11. AI architecture

Groom uses YouCam computer-vision / image-generation APIs only. **No LLM, prompt, agent, retrieval, chat or model-generated product recommendation exists in the MVP.**

Deterministic logic in Groom handles template mapping, validation, permissions, job state, task retry, cost reservation, ratings and aggregate calculations. Provider outputs are treated as untrusted binary/image output and are validated before storage/display.

No model is permitted to choose a winner or tell a brand which styles to launch. Groom presents the observed participant responses only.

---

## 12. Payments and financial logic

No payments, balances, billing or payouts in the MVP. API units are an operational usage budget, not customer money. Do not implement Stripe or payment UI.

---

## 13. Admin and operations

No public admin dashboard in the MVP. Operator tasks are performed through documented scripts/migrations and provider dashboards:
- Curate men's style templates and preview rights.
- Set/confirm provider unit budget and cost mapping.
- Monitor failed/unknown provider tasks and reconcile unit ledger.
- Inspect aggregate operational metrics (job success, latency, budget) without viewing participant images except a necessary, time-bound incident procedure.
- Disable compromised templates, pause experiment creation, rotate keys and trigger cleanup.

All operator actions affecting data/catalogue/budget should generate an `audit_events` record. MFA enabled on Supabase, Vercel, Inngest, GitHub and YouCam accounts.

---

## 14. Environment variables

| Variable | Required | Client-safe? | Purpose |
|---|---:|---:|---|
| `NEXT_PUBLIC_APP_URL` | Yes | Yes | Canonical URL for redirects/share links. |
| `NEXT_PUBLIC_SUPABASE_URL` | Yes | Yes | Supabase project URL. |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Yes | Yes | Supabase publishable/anon key; RLS remains mandatory. |
| `SUPABASE_SERVICE_ROLE_KEY` | Yes | **No** | Server privileged operations; never ship to browser. |
| `SUPABASE_INPUT_BUCKET` | Yes | No | Private source-image bucket name. |
| `SUPABASE_RENDER_BUCKET` | Yes | No | Private output bucket name. |
| `PARTICIPANT_TOKEN_HMAC_SECRET` | Yes | **No** | HMAC session token hash secret; 32+ random bytes. |
| `RATE_LIMIT_HMAC_SECRET` | Yes | **No** | HMAC key for short-lived pseudonymous rate-limit identifiers. |
| `YOUCAM_API_BASE_URL` | Yes | No | Official API host from current docs/account. |
| `YOUCAM_API_KEY` | Yes | **No** | Bearer key. |
| `YOUCAM_UNIT_BUDGET` | Yes | No | Internal allowed budget (default 700 after verifying credit). |
| `YOUCAM_HAIRSTYLE_PRESET_UNITS` | Yes | No | Current preset task cost (default 2; verify). |
| `YOUCAM_BEARD_UNITS` | Yes | No | Current beard task cost (default 2; verify). |
| `INNGEST_EVENT_KEY` | Yes | **No** | Inngest event publishing key. |
| `INNGEST_SIGNING_KEY` | Yes | **No** | Inngest request signature verification. |
| `GITHUB_REPOSITORY_URL` | Optional | Yes only if public | Actual public repository destination for footer. Empty/omitted means no GitHub footer link. |
| `PREVIEW_MODE` | Yes | No | Enables safe demo fixtures and stricter preview guards; must not bypass auth or RLS. |
| `LOG_LEVEL` | Optional | No | Defaults to info; production never logs secrets or PII. |

`NEXT_PUBLIC_` variables are bundled into client code and therefore must contain no secrets. Add `.env.example` with names and safe sample placeholders only; `.env.local` ignored by git.

---

## 15. Monitoring and operational controls

- Structured logs with `request_id`, `experiment_id`, `session_id`, `ai_job_id`, state and duration only; redact token, source bytes/URLs, signed URLs, emails and provider auth.
- Log provider task status, latency, error code and units ledger transitions without logging image data.
- Health route reports app status only. Inngest dashboard monitors durable job runs.
- Alert/notify operator (in hosting/job dashboard; no participant email) for unknown task-create outcome, budget depletion, repeated worker failures, cleanup failure and auth/security anomaly.
- Define manual kill switch `GROOM_NEW_SESSIONS_ENABLED=false` to stop new participant sessions while preserving existing sessions and cleanup.
- Define `GROOM_RENDER_ENABLED=false` to halt creation of new VTO jobs after incident.

---

## 16. Technical decisions / ADR register

| ID | Decision | Reason | Revisit only if |
|---|---|---|---|
| ADR-001 | One app, Next.js modular monolith | Simplest credible deployment. | Distinct service scaling/security need demonstrated. |
| ADR-002 | Supabase Postgres/Auth/Storage | Relational transactions, RLS, auth and private buckets. | RLS limitations appear in tested flows. |
| ADR-003 | Inngest for long-running tasks | Durable polling/retries and concurrency. | Provider supports reliable callbacks that materially simplify. |
| ADR-004 | One category per experiment | Fair comparisons and fewer provider combinations. | Operator explicitly expands after MVP validation. |
| ADR-005 | Anonymous participant with short-lived token | Low friction, minimal data. | Abuse requires account/verification. |
| ADR-006 | Aggregate-only owner results | Participant photo/vote privacy. | A separately reviewed research/privacy model authorises more. |
| ADR-007 | No LLM/payments | Not needed for core outcome. | User evidence justifies a later product phase. |
| ADR-008 | Early skeletal UI → functionality gate → approved design | Operator workflow and reduced rework. | Never waive without operator approval. |

---

## 17. Technical assumptions and Phase 0 verification blockers

Implementation must not begin beyond the skeletal shell until these are validated:
1. The YouCam API account has active key and actual redeemed units.
2. Current male hairstyle and beard template lists return usable preset IDs and permitted preview assets.
3. A real hairstyle V2.1 and beard task can be created, polled and downloaded with representative consenting test photos.
4. Provider task polling retention and required interval are known; schedule is safe for each task family.
5. Current unit cost and billing on terminal failures/timeout are confirmed in account/docs.
6. YouCam's data processing, retention, deletion, hosting and secondary-use terms are reviewed; privacy notice is accurate.
7. Supabase region and project plan, private storage/signed upload behaviour, Vercel preview protection and Inngest environment setup are available.
8. Vercel function execution limits are compatible with route work (long polling stays in Inngest, not a request handler).

If a required provider feature cannot be accessed or tested, pause and report the blocker. Do not replace real API work with a mock while claiming the MVP is functional.

---
