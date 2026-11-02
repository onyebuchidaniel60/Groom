# Groom — Application API Contract

This is the Groom application API, not the Perfect Corp API. Route handlers are thin; validate with Zod, authorise in server/domain, persist using transactions and map safe domain errors to these response formats.

## 1. Common conventions

- Base path `/api`; JSON over HTTPS except signed object upload.
- JSON request/response uses camelCase.
- UUIDs are opaque; clients must not infer sequential IDs.
- `requestId` is generated server-side and included in every error response.
- Never return raw SQL/provider errors, stack traces, owner email, participant identifiers, image paths, provider IDs/task IDs, API key or signed provider URLs.
- Owner routes require Supabase Auth session; participant routes require the `groom_participant` HttpOnly cookie, issued after consent, with `Path=/api` and 2-hour max-age.
- All mutations validate Origin/CSRF policy, Zod schema, state and ownership/session.
- All externally retried mutations are idempotent where stated.

## 2. Error envelope

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please correct the highlighted fields.",
    "requestId": "req_opaque",
    "details": [{"field": "title", "issue": "Required"}]
  }
}
```

Standard status mapping:
- `400 VALIDATION_ERROR`
- `401 AUTH_REQUIRED` or `PARTICIPANT_SESSION_REQUIRED`
- `403 FORBIDDEN` (use 404 for foreign resource IDs where specified)
- `404 NOT_FOUND`
- `409 STATE_CONFLICT`, `DUPLICATE_ACTION`, `RESPONSE_ALREADY_SUBMITTED`, `CAPACITY_REACHED`
- `410 SESSION_EXPIRED`, `EXPERIMENT_UNAVAILABLE`
- `413 FILE_TOO_LARGE`
- `415 UNSUPPORTED_FILE_TYPE`
- `422 IMAGE_NOT_ACCEPTED`, `INCOMPLETE_EVALUATION`
- `429 RATE_LIMITED`, `API_BUDGET_EXHAUSTED`
- `502 PROVIDER_UNAVAILABLE`
- `503 TEMPORARILY_UNAVAILABLE`
- `500 INTERNAL_ERROR`

## 3. Owner endpoints

### GET `/api/style-templates?type=hairstyle|beard`
Auth: Owner. Returns active manually curated `men_verified=true` catalogue rows only.

```json
{
  "data": {
    "templates": [
      {"id":"uuid","type":"hairstyle","name":"Curated style label","description":"Short factual description","previewImageUrl":null}
    ]
  }
}
```

The `previewImageUrl` is null unless a legal, stable preview asset is configured. Do not return provider template ID to browser if not required by owner UI; use internal template UUID.

### GET `/api/experiments`
Auth: Owner. Returns only owner's experiments.

```json
{"data":{"experiments":[{"id":"uuid","title":"Signature Fade","type":"hairstyle","status":"published","responseCount":8,"responseTarget":20,"createdAt":"2026-11-01T12:00:00Z"}]}}
```

### POST `/api/experiments`
Auth: Owner. Creates draft with options atomically.

Request:

```json
{
  "title":"Signature Fade Collection",
  "description":"Help us choose our next styles.",
  "participantInstructions":"Compare all previews and select one favourite.",
  "experimentType":"hairstyle",
  "responseTarget":20,
  "options":[
    {"templateId":"uuid-1","displayLabel":"Style A"},
    {"templateId":"uuid-2","displayLabel":"Style B"},
    {"templateId":"uuid-3","displayLabel":"Style C"}
  ]
}
```

Response `201`:

```json
{"data":{"experiment":{"id":"uuid","title":"Signature Fade Collection","experimentType":"hairstyle","status":"draft","responseTarget":20,"options":[{"id":"uuid","displayLabel":"Style A"}]}}}
```

### GET `/api/experiments/:id`
Auth: Owner. Owner-scoped detail, options, state, safe response count and timestamps. No participant-level values.

### PATCH `/api/experiments/:id`
Auth: Owner. Draft only. Editable fields: title, description, participant instructions, response target and options. Reject `ownerId`, `status`, `publicSlug`, counters, result values, created/updated timestamps or provider fields. Response schema matches safe detail.

### POST `/api/experiments/:id/publish`
Auth: Owner. Draft with 3–4 active verified templates.

Response `200`:

```json
{"data":{"status":"published","publicUrl":"https://<canonical-app-host>/p/<random-slug>"}}
```

Canonical host must come from `NEXT_PUBLIC_APP_URL` and be validated against deployment environment. Slug is generated server-side.

### POST `/api/experiments/:id/pause`
Auth: Owner. Published → paused. Returns `{data:{status:"paused"}}`.

### POST `/api/experiments/:id/resume`
Auth: Owner. Paused → published only if date/target/capacity rules allow. Returns `{data:{status:"published"}}`.

### POST `/api/experiments/:id/close`
Auth: Owner. Published/paused → closed terminal. Returns `{data:{status:"closed"}}`.

### POST `/api/experiments/:id/archive`
Auth: Owner. Closed → archived. Returns `{data:{status:"archived"}}`.

### GET `/api/experiments/:id/results`
Auth: Owner. Returns aggregate-only response. Below 5 responses:

```json
{"data":{"resultsAvailable":false,"completedResponses":4,"responseTarget":20,"unlockAt":5}}
```

At or above 5:

```json
{
  "data": {
    "resultsAvailable":true,
    "completedResponses":12,
    "options":[
      {"optionId":"uuid","label":"Style A","averageRating":4.2,"ratingDistribution":{"1":0,"2":1,"3":2,"4":3,"5":6},"firstChoiceCount":7,"firstChoiceShare":58}
    ],
    "segments":[{"name":"wavy","count":10,"options":[{"optionId":"uuid","averageRating":4.1,"firstChoiceShare":50}]}],
    "limitations":["Results reflect only people who participated and are directional, not statistically representative."]
  }
}
```

Do not include segment entries with count <10. No individual response IDs, participant data, session identifiers or render URLs.

## 4. Public participant endpoints

### GET `/api/public/experiments/:slug`
Public. Returns only currently accepting study details.

```json
{"data":{"title":"Signature Fade Collection","description":"...","participantInstructions":"...","businessName":"Example Barbers","experimentType":"hairstyle","optionCount":3,"status":"published"}}
```

No owner email, internal experiment UUID, provider template IDs, results, counts by segment or participant records.

### POST `/api/public/experiments/:slug/sessions`
Public + rate limit. Requires explicit consent and 18+ self-attestation. One active participant cookie at a time.

Request:

```json
{"idempotencyKey":"random-client-generated-uuid","consentAccepted":true,"ageConfirmed":true,"hairTexture":"wavy"}
```

`hairTexture` is accepted only for hairstyle experiments and is optional. Response `201` sets `groom_participant` cookie (`HttpOnly; Secure; SameSite=Lax; Path=/api; Max-Age=7200` in production) and returns:

```json
{"data":{"status":"consented","expiresAt":"2026-11-01T14:00:00Z","nextStep":"upload"}}
```

Server derives the opaque token with HMAC from the random session ID, experiment ID and idempotencyKey, stores only a purpose-separated HMAC hash, and can re-derive the same token for an idempotent retry. Cookie is cleared on submit/expiry. Cookie deletion/incognito can create another participant session; Groom does not claim identity-based one-person-one-vote enforcement.

### POST `/api/participant/upload-url`
Participant cookie. Request:

```json
{"contentType":"image/jpeg","fileSize":2345678}
```

Response:

```json
{"data":{"uploadUrl":"<short-lived-supabase-signed-upload-url>","uploadToken":"opaque-one-time-token","expiresAt":"2026-11-01T12:05:00Z"}}
```

The signed Supabase upload URL is a temporary credential; do not log it. Storage path is generated by server and not chosen by client.

### POST `/api/participant/upload-complete`
Participant cookie. Request:

```json
{"uploadToken":"opaque-one-time-token"}
```

Server verifies object, validates actual bytes, normalises and creates AI jobs idempotently. Response:

```json
{"data":{"status":"rendering","completedOptions":0,"totalOptions":3}}
```

### GET `/api/participant/session`
Participant cookie. Returns own session status and signed private result URLs only when session `ready`.

```json
{"data":{"status":"rendering","jobs":[{"status":"running"},{"status":"succeeded"},{"status":"queued"}],"expiresAt":"2026-11-01T14:00:00Z"}}
```

The response must not expose provider task IDs, raw provider error, source object path or provider signed URLs. For ready session, include option IDs, labels and short-lived signed URLs for previews in stored random display order.

### POST `/api/participant/submit`
Participant cookie; idempotent by session + request key. Request:

```json
{
  "idempotencyKey":"client-generated-random-uuid",
  "firstChoiceOptionId":"uuid-1",
  "ratings":[
    {"optionId":"uuid-1","rating":5},
    {"optionId":"uuid-2","rating":3},
    {"optionId":"uuid-3","rating":4}
  ]
}
```

Response:

```json
{"data":{"status":"submitted","message":"Thank you. Your response has been recorded."}}
```

Repeated identical submit returns same success state. Conflicting payload after submission returns `409 RESPONSE_ALREADY_SUBMITTED`. After commit, schedule deletion of photo/render objects and clear cookie.

## 5. Inngest contract

- `/api/inngest` is protected by the official Inngest signing mechanism.
- Event payloads include only internal `sessionId`/`jobId` and event version; worker reloads authoritative rows from DB.
- Event retries are expected. Database unique constraints and persisted provider task ID determine whether work is resumed or new work is allowed.
- No provider secret or personal photo is included in event payload.

## 6. API versioning

This hackathon MVP does not expose a public versioned API to third parties. Internal app routes are `/api/...`; breaking internal changes are coordinated across frontend and tests. If a public integration API is introduced later, add `/api/v1` and a separate auth/rate-limit/version policy after operator approval.
