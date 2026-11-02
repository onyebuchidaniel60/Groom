# GROOM — PROJECT SPECIFICATION

**Document status:** Blueprint v1.0 — proposed source of truth; operator approval required before implementation  
**Product:** Groom  
**Competition context:** YouCam API Skin AI & eCommerce VTO Hackathon  
**Prepared:** 1 November 2026  
**Implementation status:** Not started. This document is a specification, not application code.

---

## 1. Product definition

### 1.1 One-line description

**Groom is a virtual consumer-testing platform that lets men's grooming brands and barbershop businesses compare proposed hairstyle or beard-style collections on consenting participants before deciding which styles to launch, promote or stock.**

### 1.2 Problem

A grooming business planning a new haircut or beard-style collection often makes selection decisions using internal taste, informal social-media polls, competitor observation or physical focus groups. Those methods may not show how the candidate styles look on the actual people being asked to evaluate them. Groom creates a structured, image-based preference exercise: each participant sees the same proposed alternatives applied to their own photo, evaluates each option and chooses one favourite. The business receives aggregated preference results.

**Evidence status:** This is a plausible problem hypothesis, not a validated market-demand or willingness-to-pay claim. Existing virtual try-on and consumer-research services establish adjacent categories, not demand for Groom specifically. The concept's originality is **NOT READY / unproven** pending further prior-art research. Do not claim Groom is globally novel.

### 1.3 Current alternatives / workarounds

- Informal polls, Instagram stories, WhatsApp groups and customer conversations.
- Physical focus groups and sample photography.
- Barber or brand staff selecting styles based on experience.
- Existing virtual hairstyle/beard try-on products used individually, without necessarily producing a controlled, multi-participant brand experiment.
- Conventional market-research agencies and consumer panels.

Groom's proposed distinction is a repeatable brand-owned experiment: one collection, a fixed candidate set, the same set of options rendered on each participant, structured ratings and a single first-choice signal, followed by aggregate-only reporting. This differentiation is a hypothesis and must be challenged against current products before any public novelty claim.

### 1.4 Target users

**Primary buyer / experiment owner**
- Men's grooming brands testing a proposed haircut or beard-style collection.
- Barbershop chains or independent barbers testing a signature menu or seasonal collection.
- Grooming-product/service businesses conducting a small pre-launch preference study.

**Participant**
- Adult men invited through a public experiment link to compare a limited set of virtual styles.

**Operator**
- Groom project operator with infrastructure access. No separate in-product super-admin console is required for the hackathon MVP.

### 1.5 User roles

| Role | How they authenticate | Permissions |
|---|---|---|
| Business owner | Supabase email/password account | Create/manage own experiments; publish, pause, resume, close and archive; see aggregate results for owned experiments. |
| Participant | Anonymous, short-lived browser session | Consent, upload one photo, view own generated previews, rate options and submit once. No account. |
| Platform operator | Out-of-band infrastructure credentials only | Manage template catalogue, API budget, deployment and incident response. No general participant-photo browsing workflow. |

There are no team seats, collaborators or participant accounts in the MVP.
Anonymous participation means Groom can enforce one response per issued browser session, not one response per human. Clearing cookies or switching browsers can create another session; this is a known anti-fraud limitation, not identity verification.

### 1.6 Core value proposition

A grooming business can collect structured, visual preference feedback on proposed styles before committing to a launch or promotion, without running a full research programme or asking consumers to imagine a style from a text description alone.

### 1.7 Key differentiator hypothesis

The proposed difference is **not** “AI haircut try-on.” It is a controlled pre-launch experiment that:
1. Defines one grooming collection and 3–4 candidate styles.
2. Applies every candidate to the same participant photo using the relevant YouCam feature.
3. Randomises presentation order to reduce position bias.
4. Captures a rating for every option and exactly one first-choice vote.
5. Gives the owner aggregate-only comparison results, with small-segment suppression.

This mechanism is still subject to originality validation. Do not describe it as unique, first-of-its-kind or proven to improve launch outcomes.

### 1.8 Primary use case

A barbershop chain creates a “New Signature Fade Collection” hairstyle experiment with four curated styles, publishes a share link, invites adult customers, receives virtual previews and sees which candidate is most preferred overall and, when enough observations exist, across self-reported hair-texture segments.

### 1.9 Secondary use cases

- A men's grooming brand tests four beard styles before selecting a campaign look.
- A barbershop tests a new haircut menu before rolling it out across branches.
- A grooming brand compares a small seasonal style collection for content or merchandising decisions.

Secondary use cases must use the same experiment workflow; they do not add new product modules.

### 1.10 Explicit non-goals

Groom is **not**:
- A barber booking, appointment, salon-management or payments platform.
- A consumer social network, public style gallery or influencer marketplace.
- A personalised haircut recommender or an AI stylist/chatbot.
- A face-recognition, identity-verification, biometric-identification or demographic-inference product.
- A clinical, medical, dermatological or hair-health diagnostic tool.
- A guarantee of how a haircut or beard will look after a real barber performs it.
- A physical product fit, hair-growth or service-quality predictor.
- A representative national market survey or statistically certified research panel.
- A platform for children; participants must affirm they are at least 18.
- A platform for uploading arbitrary reference images in the MVP.

---

## 2. MVP scope

### 2.1 Fixed MVP decision

Each experiment tests **one category only**:
- `hairstyle`, or
- `beard`.

A single experiment must never mix hairstyle and beard transformations. This avoids combining separate provider models and makes the comparisons easier to interpret. The product supports both categories, but not in the same experiment.

### 2.2 MUST HAVE

1. Owner registration, sign-in, sign-out and protected dashboard.
2. Owner profile with business name and business type (`brand`, `barbershop`, `other`).
3. Curated YouCam template catalogue, separated into hairstyle and beard categories; only manually verified men's templates are selectable.
4. Create an experiment with title, short description/instructions, one category, response target and 3–4 selected styles.
5. Draft, publish, pause, resume, close and archive lifecycle.
6. High-entropy public share URL for a published experiment.
7. Public experiment landing page and participant consent / 18+ affirmation.
8. Anonymous participant session with secure, short-lived cookie.
9. One photo upload, basic image guidance, server-side image validation/normalisation.
10. Generate one YouCam preview per selected style using the correct API family.
11. Durable task execution and polling; participant sees progress and recoverable failure states.
12. Display all completed previews in a randomised but stable order.
13. Participant rates every option from 1–5 and selects exactly one favourite.
14. One final submission per participant session; duplicate submissions prevented.
15. Owner dashboard with aggregate response count, average rating, rating distribution, first-choice count/share and completion/failure counts.
16. Suppress aggregate results until at least 5 completed responses; suppress segment breakdowns when segment sample size is below 10.
17. Private object storage, retention/deletion jobs, audit events and unit-usage ledger.
18. README and `/docs` route plus repository documentation.
19. Early skeletal frontend deployment and all staged gates defined in `IMPLEMENTATION_PLAN.md`.

### 2.3 SHOULD HAVE (only after all MUST HAVE behaviour is verified)

- Optional self-reported hair texture for hairstyle experiments: `straight`, `wavy`, `curly`, `coily`, `unsure`, `prefer_not_to_say`. It is not inferred by AI. Segment results only appear at N ≥ 10.
- CSV export of the same aggregate data shown in the dashboard. Do not export participant-level rows.
- Copy-link control with clear copied/error feedback.

### 2.4 NICE TO HAVE

- Owner logo/brand accent on public experiment page.
- Optional experiment end date.
- Aggregate result snapshot suitable for presentation.

These are not schedule-critical. Do not implement them before the MUST HAVE acceptance tests pass.

### 2.5 FUTURE (not part of hackathon MVP)

- Multiple team members and brand workspaces.
- Paid plans, invoicing and payment processing.
- Larger or recruited consumer panels.
- Additional hair, nail, makeup or accessory categories.
- Physical product tests, barber scheduling or commerce integrations.
- A/B test design, statistical confidence intervals or professional research exports.
- AI-assisted experiment design or recommendations.
- Custom style reference-image upload.

### 2.6 OUT OF SCOPE

Payments, subscriptions, booking, chat, comments, DMs, LLMs, recommendation models, public profile pages, participant accounts, email campaigns, arbitrary image URLs, social login, mobile native apps, multi-tenant team permissions, individual-vote exports and any face identity matching.

### 2.7 MVP limits

| Limit | Rule |
|---|---|
| Options per experiment | Minimum 3, maximum 4 |
| Active published experiments per owner | 3 |
| Response target | Default 20; allowed range 10–50 |
| Experiment category | Exactly one: hairstyle or beard |
| Participant | One active session per browser cookie at a time; no account |
| Session TTL | 2 hours from creation |
| Photo input | One JPEG/JPG, max 10 MB before normalisation |
| Results unlock | 5 completed participants |
| Segment display | Segment count must be at least 10 |
| Global MVP API budget | Internal cap 700 units, configurable; retain 300 units from the initial 1,000-unit credit as contingency only if the account actually has that credit |
| API unit estimate | 2 units per successful-style task as currently documented; 6–8 units per complete participant for 3–4 options. Reconfirm in live account before launch. |

The global 700-unit cap is a local safety budget, not a claim about the provider's actual remaining balance. The system must never assume a 1,000-unit credit has been redeemed.

---

## 3. User flows

### 3.1 Owner registration and onboarding

**Entry:** `/sign-up`  
**Preconditions:** None.  
**Steps:** Enter email and password → validate → create Supabase Auth user → create `profiles` row → verify email if enabled by deployment settings → redirect to `/dashboard`.  
**State changes:** Auth user and profile inserted.  
**Success:** Dashboard with empty state and “Create experiment” action.  
**Failures:** Existing email, weak password, provider/network error, profile creation failure. If profile insertion fails after auth creation, retry idempotently on next authenticated request; do not create duplicate profile.  
**Permissions:** Authenticated owner only.  
**Exit:** Dashboard.

### 3.2 Create experiment

**Entry:** `/dashboard/experiments/new`.  
**Preconditions:** Signed-in owner; fewer than 3 active published experiments.  
**Steps:** Enter title, description and participant instructions → choose `hairstyle` or `beard` → fetch active verified template catalogue for that category → select 3 or 4 distinct templates → set response target (10–50; default 20) → save draft.  
**System:** Validate category/template match, unique template IDs, supported male catalogue flag and target. Insert `experiments` and `experiment_options` in one transaction.  
**Success:** Draft detail page.  
**Failure/edge:** Template was deactivated while editing; duplicate option; category mismatch; owner active-experiment limit; DB failure. Return field-specific errors; no partial experiment.  
**Exit:** Draft detail page.

### 3.3 Publish experiment

**Entry:** Draft detail page.  
**Preconditions:** Owner owns experiment; status `draft`; exactly 3–4 valid options; title/instructions valid; no provider/API configuration blocker.  
**Steps:** Owner clicks Publish → confirmation summarises category, options and response target.  
**System:** Transaction locks experiment; rechecks ownership/status/options/templates; generates random public slug; sets status `published`, `published_at`; records audit event. After publish, category and options are immutable.  
**Success:** Public share link is displayed.  
**Failure:** Already published, invalid catalogue, slug collision (regenerate), owner limit, database error.  
**Permissions:** Owner only.  
**Exit:** Experiment detail.

### 3.4 Public experiment discovery

**Entry:** `/p/{slug}`.  
**Preconditions:** Experiment is `published`, within date window and below response target.  
**System:** Resolve slug; return only public-safe fields (title, public description/instructions, owner business display name, category, option count and privacy links). Do not return owner email, internal IDs, API identifiers, response data or template provider credentials.  
**Success:** Participant sees intro and start button.  
**Failure:** Unknown slug → not found; paused/closed/archived/target reached → unavailable screen with appropriate message.  
**Exit:** Start participant session.

### 3.5 Participant consent and session creation

**Entry:** Public experiment page.  
**Preconditions:** Experiment accepts new participants.  
**Inputs:** `idempotencyKey` UUID; explicit unchecked consent checkbox; separate 18+ self-attestation; optionally hair texture self-report for hairstyle study. Draft consent: “I agree to Groom using my photo to generate the virtual grooming previews and sending it to Perfect Corp/YouCam for that processing. The business running this study will receive aggregate preference results, not my photo or individual vote. Groom keeps its copy temporarily for processing and deletes it under the stated retention schedule. See the Privacy Notice for third-party processing details.” Final wording/version must be checked against provider terms and reviewed before public use.  
**System:** Validate affirmative consent and age affirmation; derive an opaque 256-bit participant token using HMAC-SHA256 over the random session UUID, experiment UUID and client `idempotencyKey` with a server secret; persist only a separately purpose-derived HMAC token hash; set HttpOnly, Secure (production), SameSite=Lax cookie named `groom_participant` with `Path=/api`; 2-hour expiry; record consent version/time. This deliberately supports one active participant session per browser at a time. If a valid active participant cookie already exists, reject a new session until the prior session is submitted or expires; if prior session is terminal, clear cookie and allow a new one. No name/email required.  
**Success:** Photo capture/upload screen.  
**Failure:** Consent absent, under-18 answer, expired/unavailable experiment, cookie disabled, rate limit. No image upload occurs before consent.  
**Permissions:** Session token authorises only that participant session and its experiment.  
**Exit:** Upload screen.

### 3.6 Photo upload and validation

**Entry:** Participant upload screen.  
**Preconditions:** Valid participant cookie/session in `consented` state.  
**Instructions:** One adult male face, facing camera, clear full face and hair/beard area, good lighting, no additional people, no sunglasses/face covering, shoulders visible for hairstyle, no filters. The stricter provider-specific checks are applied by category.  
**Steps:** Browser requests signed upload URL → uploads file bytes directly to private Supabase bucket → calls upload-complete.  
**System:** Check MIME by file signature, size, image decodability, dimensions and pixel cap; use `sharp` to re-encode JPEG, resize to max 1024 long side and strip EXIF. Groom does not implement a separate face/pose detector; YouCam validates face landmarks/pose and returns a safe error that prompts re-upload. Never trust browser-provided MIME or filename. Store only random storage path.  
**Success:** Input stored; session moves to `uploaded`; enqueue one render job per option.  
**Failure:** Invalid/corrupt image, unsupported format, dimensions, upload expiry, storage outage, experiment paused/closed, session expired. Allow one clean re-upload before rendering; old object is deleted.  
**Privacy:** No public URL; no raw image URL passed back to owner.

### 3.7 AI generation and polling

**Entry:** Enqueued `groom/render-requested` event.  
**Preconditions:** Valid session, consent, source image, published experiment, option template still active, unit budget reserved.  
**System:** One job per option. Inngest worker acquires concurrency slot, creates YouCam upload (`POST /s2s/v2.0/file`), uploads bytes to returned signed PUT URL, then creates appropriate VTO task. Hairstyle uses `/s2s/v2.1/task/hair-transfer`; beard uses `/s2s/v2.0/task/beard-style`. Persist provider task ID immediately. Poll according to verified provider retention window; store the returned image in private storage.  
**Success:** All option jobs `succeeded`; session becomes `ready`.  
**Failure:** Mark per-option `failed`; retry only transient polling/download/storage issues with bounded attempts. Do not blindly repeat an ambiguous task-creation POST; mark `unknown_create_outcome` for operator review to prevent duplicate billing. If one option permanently fails, do not allow a partial or unfair comparison; show recoverable state and allow one controlled rerun only after verifying no prior task is still live and unit accounting permits it.  
**Participant view:** Progress by option, not provider technical details. No preview result is shown until all selected options are successful.

### 3.8 Submit evaluation

**Entry:** Ready results screen.  
**Preconditions:** Session is `ready`; every option has a successful render; no prior response.  
**Inputs:** One rating 1–5 per option and exactly one first-choice selection.  
**System:** Validate all option IDs belong to experiment, exactly one first choice, all ratings present, session token valid and unexpired. In one DB transaction, insert response and ratings, mark session submitted, reserve response slot / increment count atomically, write audit event, then schedule render/input deletion.  
**Success:** Thank-you confirmation; no more editing or second submission.  
**Failure:** Invalid rating, missing option, duplicate request, response target reached, session expired, transaction conflict. If retry after timeout, return existing successful response result based on idempotency key/session state; never insert twice.  
**Exit:** Thank-you page.

### 3.9 Owner results

**Entry:** `/dashboard/experiments/{id}/results`.  
**Preconditions:** Signed-in owner owns experiment.  
**System:** Compute aggregate data on server or via restricted SQL view. If <5 completed responses, show response count and “results unlock after 5 completed responses,” but no rating/vote aggregate. Hide any segment with n<10. Never return individual ratings or identifiers.  
**Outputs:** Completed count, target progress, average rating per option, rating distribution (1–5), first-choice count/share per option, failure/abandonment counts in aggregate. Hair-texture segments only if the segment is >=10.  
**Failure:** Unauthorized → 404/403 policy; incomplete data; DB failure with retry state.  
**Exit:** Experiment detail or dashboard.

### 3.10 Pause, resume, close, archive

- **Pause:** Owner temporarily stops new participant sessions; existing unexpired sessions may finish.
- **Resume:** Owner reopens new sessions if target/date rules permit.
- **Close:** Permanently stop new sessions; existing sessions may finish until expiry; cannot reopen.
- **Archive:** Owner hides closed/past experiment from active list; aggregate results remain available. Cannot unarchive in MVP.

Every transition is owner-authorised, transactional, validated against the state machine and audited.

---

## 4. Functional requirements and acceptance criteria

### FR-01 Authentication
- Actor: Owner.
- Preconditions: Valid email and password.
- Processing: Supabase Auth; create profile idempotently.
- Acceptance: Owner can sign up, sign in, sign out and refresh a protected route; unauthenticated access redirects; one owner cannot read another owner's experiments.

### FR-02 Curated template catalogue
- Actor: Owner.
- Input: Category query.
- Processing: Return active, manually approved male templates only. Provider template IDs are seeded from real API responses; no guessed IDs.
- Acceptance: Hairstyle list contains only verified male presentation templates; beard list only verified men's beard styles; no inactive or mismatched template can be saved.

### FR-03 Experiment creation
- Actor: Owner.
- Input: Title (3–80 chars), description (optional, max 500), instructions (max 500), one category, 3–4 distinct template options, response target 10–50.
- Acceptance: Valid draft persists; invalid fields have clear errors; category and options save atomically; owner cannot exceed 3 active published studies.

### FR-04 Publish lifecycle
- Actor: Owner.
- Preconditions: Valid draft.
- Acceptance: Publish generates unique cryptographically random slug and public URL; published options/category immutable; only owner can pause/resume/close/archive; invalid transitions rejected.

### FR-05 Public experiment page
- Actor: Participant.
- Acceptance: Page renders only approved public fields; invalid slug is not found; non-published state never allows a new session; page works on mobile and desktop.

### FR-06 Consent and anonymous session
- Actor: Participant.
- Acceptance: No session/upload before affirmative consent and 18+ self-attestation; age is not identity-verified; cookie is HttpOnly, Secure in production, SameSite=Lax; only token hash persisted; session expires after 2 hours; participant cannot access another session by changing IDs.

### FR-07 Upload
- Actor: Participant.
- Acceptance: Only valid single JPEG/JPG under 10MB; server validates bytes and normalises to JPEG <=1024 long side; EXIF stripped; object is private; no client-supplied storage key; no raw public URL returned.

### FR-08 Preview generation
- Actor: System.
- Acceptance: Each selected option creates exactly one persisted AI job; provider upload bytes actually reach the signed URL; correct category endpoint is used; task ID stored before polling; output copied to private storage; status updates observable; no unbounded poll or duplicate task creation.

### FR-09 Evaluation
- Actor: Participant.
- Acceptance: All option previews present; display order stable across refresh; participant must provide one rating for every option and exactly one favourite; one response per session; duplicate submit is idempotent; aggregate count increases once.

### FR-10 Results
- Actor: Owner.
- Acceptance: Owners see only own experiments; fewer than 5 responses suppress aggregates; segment size below 10 suppressed; no individual response, photo, token or render path appears in network response; calculations match seeded test fixtures.

### FR-11 Retention and deletion
- Actor: System.
- Acceptance: Source input is deleted after all AI jobs are terminal and verified to no longer need it; renders deleted at submit/expiry; expired participant sessions cleaned at least hourly; deletion failures alert/log and retry; owner cannot browse image history. Vendor-side deletion/retention claims are not made until Perfect Corp terms are checked.

### FR-12 Documentation
- Actor: Developer/operator.
- Acceptance: `README.md`, `docs/` setup/configuration/testing/deployment/troubleshooting, privacy explanation and user guide exist. `/docs` is reachable. Homepage footer uses real `/docs` and actual public GitHub URL when available; no fabricated URLs.

---

## 5. Business logic and state machines

### 5.1 Experiment states

```text
draft ──publish──> published ──pause──> paused
  │                   │                  │
  │                   ├──close───────────┼──> closed
  │                   └──target reached──┘
  │                                      │
  └──discard (future only)               └──resume──> published (only if not closed)
closed ──archive──> archived
```

Allowed transitions:
- `draft -> published`
- `published -> paused | closed`
- `paused -> published | closed`
- `closed -> archived`
- No transitions out of `archived`.
- Experiment category/options editable only in `draft`.
- Closing an experiment is terminal; archive is a visibility state, not deletion.

### 5.2 Participant session states

```text
created -> consented -> uploaded -> rendering -> ready -> submitted
    |          |            |          |           |
    └──────────┴────────────┴──────────┴──────────> expired
                                      └───────────> failed
```

- `created` is inserted only after consent and age affirmation.
- `consented` means consent recorded; no image yet.
- `uploaded` means validated image stored.
- `rendering` means render jobs enqueued/running.
- `ready` means all option renders succeeded.
- `submitted` is terminal.
- `failed` is terminal after unrecoverable failure or operator abort.
- `expired` is terminal after two hours.

### 5.3 AI job states

```text
queued -> unit_reserved -> file_uploaded -> task_created -> running -> succeeded
   |            |              |                 |             |
   └────────────┴──────────────┴─────────────────┴────────────> failed
                                           └──────────────────> unknown_create_outcome
```

The provider may charge for a task that times out before status is polled. Never automatically repeat a task-create POST when the outcome is ambiguous. Persist correlation details and require operator review.

### 5.4 Cost and quota rules

- Documented cost for selected preset hairstyle V2.1 and beard V1.0 operations is currently 2 units per task; re-confirm before use.
- Estimated participant cost is 6 units for 3 options, 8 units for 4.
- Reserve units before provider task creation using a DB transaction/atomic budget update.
- Mark a ledger row `committed` when provider work is confirmed billable/successful according to verified billing semantics; release only when provider docs/account confirm no charge. Unknown outcome remains reserved/unknown until reconciliation.
- Internal global budget defaults to 700 units; operator may change only after checking actual provider account. Do not show vendor balance as live unless fetched from an official endpoint.
- Stop accepting new sessions when estimated budget is exhausted; existing jobs are allowed to finish within reserved units.

### 5.5 Response and ranking logic

- Every participant rates every option with integer 1–5.
- Exactly one option is the participant's first choice.
- Option order is randomised per session and stored as UUID array; refresh must preserve it.
- `first_choice_share = first_choice_count / completed_response_count * 100`.
- `average_rating = sum(ratings_for_option) / count(ratings_for_option)`.
- Report count and average to one decimal; share to whole percent. Never describe these small-panel values as statistically representative.
- Do not rank segments when segment count is under 10.
- Ties remain ties; do not invent a tie-break score.

### 5.6 Completion target and concurrency

- Response target is 10–50.
- Session creation increments an atomic `active_or_completed` reservation so concurrent requests cannot overbook the target. A participant session holds a slot for 2 hours; expired/failed sessions release it.
- The completed response count increments exactly once on submit.
- If target reached, status becomes `closed` automatically after the final accepted submit. Sessions already created remain valid until expiry and can submit only if the reserved slot is still valid.
- If an experiment is paused/closed, existing sessions may complete; new sessions are blocked.

### 5.7 Duplicate-prevention rules

- Unique owner + public slug.
- Unique experiment + template ID among options.
- Unique participant session token hash.
- Unique `response.session_id`.
- Unique `(response_id, option_id)` rating.
- Unique provider task id when present.
- Unique idempotency key per participant action.
- Every write checks the authenticated owner or session token server-side; UI hiding is not authorisation.

### 5.8 Expiration / cancellation

- Participant session expires 2 hours after creation; after expiry, no submit or render fetch permitted.
- A participant can abandon and close the tab; cleanup removes their stored images at expiry.
- Owner can pause or close an experiment; this blocks new participant starts but does not terminate valid already-started sessions.
- No owner delete endpoint in MVP; archive is sufficient. Physical deletion is operator-led retention workflow.

---
