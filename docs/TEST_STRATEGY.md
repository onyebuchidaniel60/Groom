# Groom — Test Strategy and Acceptance Matrix

## 1. Test pyramid

- **Unit (Vitest):** pure validation, state machines, scoring/aggregation, cost estimates, idempotency helpers, image-policy helpers.
- **Integration (Vitest + isolated Supabase):** RLS, DB transactions, ownership, constraints, storage signing, repository functions, API handlers, worker persistence.
- **Provider contract tests:** Zod-validate redacted current YouCam response fixtures; run minimal live hairstyle and beard smoke tasks only with operator-approved test images/keys.
- **Component tests:** React Testing Library for forms, consent, ratings, upload states and result suppression.
- **E2E (Playwright):** real deployed Preview; separate owner accounts and isolated test studies. Do not use production participant photos.
- **Security tests:** negative tests for IDOR, cookie manipulation, RLS bypass, signed URL leaks, malicious uploads, CSRF, replay and concurrent submits.

## 2. Unit test cases

### Experiment validation
- Reject title shorter than 3 or longer than 80.
- Reject fewer than 3 or more than 4 options.
- Reject duplicate template IDs.
- Reject options from a different experiment category.
- Reject inactive or not-men-verified template.
- Reject response target outside 10–50.
- Reject any attempt to edit category/options after publish.

### State machines
- Verify every allowed experiment transition.
- Reject draft→paused, closed→published, archived→anything.
- Reject new session when paused/closed/archived/target reached/outside date window.
- Permit already-issued, unexpired participant session to finish after pause/close as specified.
- Verify participant session terminal states cannot be reopened.
- Verify failed/expired session does not free response capacity twice.

### Evaluation / aggregation
- Ratings must be integer 1–5.
- Exactly one first choice.
- Each option rated exactly once.
- Average rating and distribution calculations match fixtures.
- First-choice share denominator equals submitted responses only.
- No results before 5 completed responses.
- No segment result when segment N<10; show at N=10.
- Ties remain ties.

### Unit accounting
- 3 options at 2 units reserves 6; 4 options reserves 8.
- Reject reservation exceeding budget.
- Concurrent reservations cannot overdraw budget.
- Successful billed task moves reserved→committed once.
- Provider-confirmed non-billable terminal failure releases once.
- Unknown task outcome remains reserved/unknown and is not auto-retried.

## 3. Integration tests

### Authentication / RLS
- User A can create/read/update own draft.
- User A cannot read/update/delete User B's experiment by guessing UUID.
- Anonymous cannot read `respondent_sessions`, `ai_jobs`, `responses`, `response_ratings`, `usage_ledger` or `audit_events` tables.
- Owner cannot select individual response rows or participant image paths.
- Public routes return only safe experiment/template fields.

### Upload / storage
- Signed upload URL only authorises intended bucket/path and expires.
- Invalid MIME header with non-JPEG bytes rejected.
- Corrupt JPEG rejected.
- Oversized file rejected before expensive processing.
- EXIF removed and output JPEG is <=1024px long side.
- Private bucket cannot be fetched without short-lived signed URL.
- Upload completion called twice produces one set of AI jobs.

### YouCam / jobs
- File API response must contain file ID and signed PUT URL.
- Verify byte upload to signed PUT URL actually completes before task POST.
- Hairstyle jobs use V2.1 endpoint and correct preset ID.
- Beard jobs use V1.0 endpoint and correct preset ID.
- Poll response states are mapped to internal job states; unknown provider state fails closed.
- Worker retry resumes known `provider_task_id` rather than creating a new task.
- Timeout before receiving task ID results in `unknown_create_outcome` and no second POST.
- Result download rejects unapproved hosts, non-HTTPS, redirects to private IP and unexpected content type/size.
- Result file stored in private bucket; provider signed URL is never returned to client or logged.

### Transactions / concurrency
- Two concurrent publishes cannot exceed owner limit.
- Two concurrent session starts at response target reserve only allowed slots.
- Two simultaneous submit requests produce one response and one count increment.
- Two simultaneous unit reservations cannot exceed budget.
- Duplicate Inngest events do not duplicate AI jobs or tasks.

## 4. E2E scenarios (must pass on deployed Preview)

### E2E-01 Owner happy path
1. Owner signs up and signs in.
2. Dashboard empty state appears.
3. Owner creates a hairstyle study with three verified templates and target 10.
4. Save draft, reopen, publish.
5. Public link is created and opens in private browser.
6. Owner can pause/resume/close; invalid transitions are not offered or rejected.

### E2E-02 Participant hairstyle happy path
1. Open public share link.
2. Accept consent and affirm 18+.
3. Upload approved test selfie.
4. See processing for three options.
5. Receive all real YouCam hairstyle outputs.
6. Rate every option 1–5 and select exactly one favourite.
7. Submit and see thank-you.
8. Refresh/back/replay request cannot submit a second time.
9. Stored input/render objects are removed according to retention workflow.

### E2E-03 Participant beard happy path
Same as E2E-02, but beard study and beard API. Verify stricter face width/yaw/photo requirements and error guidance.

### E2E-04 Results and privacy
1. With four responses, owner sees count and unlock notice but no aggregates.
2. After fifth response, owner sees aggregate ratings and first-choice totals.
3. A segment with nine responses is suppressed; at ten it may be shown.
4. Owner cannot request individual response/session/render endpoints.

### E2E-05 Lifecycle / expiry
- Paused and closed studies reject new starts.
- A session started before pause can finish before expiry.
- Expired session cannot retrieve previews or submit.
- Reached target closes study and blocks new participants.

### E2E-06 Negative security
- User A cannot read user B's study with changed path ID.
- Participant A cannot retrieve Participant B's session by changing request body/URL.
- Missing consent or age affirmation prevents session creation/upload.
- Arbitrary external source URL cannot be submitted.
- No secret appears in JS bundle or browser network response.

## 5. Acceptance gate before visual design

Final visual design must not begin until:
- E2E-01 through E2E-06 pass on deployed Preview, except an explicitly documented test dependency that the operator approves as an exception.
- At least one real hairstyle provider task and one real beard provider task complete end-to-end with safe test images.
- All critical owner/participant permissions pass.
- Unit budget/cost ledger is reconciled.
- No high-severity open defects in experiment lifecycle, consent, photo handling, rendering, voting or results.
- Operator receives exact Preview URL, deployed commit SHA, test report and known limitations.

## 6. Release acceptance

Run lint, typecheck, unit, integration, E2E, production build, dependency/security audit, migration dry run, storage cleanup verification, link check, responsive checks and accessibility checks. Production deployment is a separate operator approval gate.
