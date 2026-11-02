# GROOM — IMPLEMENTATION PLAN

**Status:** Approved sequence proposed by blueprint; operator approval required before implementation.  
**Execution rule:** One assigned task/phase at a time. Implement → test → inspect → fix → commit → checkpoint → stop.

The plan follows the attached **Proper Vibe Coding Workflow**. See the source methodology at `turn2file0` / the user-provided `PROPER_VIBE_CODING_WORKFLOW_REVISED(4).md`.

---

## Phase 0 — Repository, accounts and critical integration reconnaissance (read-only except test assets)

**Objective:** Confirm the project environment and remove fatal assumptions before building more than the skeletal frontend.  
**Dependencies:** Operator provides/creates GitHub repository and accounts; no production data.  
**Tasks:**
1. Inspect repository, package manager, Node version, existing source, docs, environment variable names, Vercel project and Supabase projects. Do not overwrite or re-scaffold blindly.
2. Confirm YouCam API account, Bearer key availability and actual remaining units/credit redemption.
3. Call official hairstyle V2.1 and beard template endpoints. Save a redacted template catalogue snapshot; curate 3–4 male templates per category. Verify each output visually and preview-asset usage terms.
4. Using consenting test photos, run one hairstyle preset and one beard task end-to-end: File API → PUT image bytes → task POST → poll → download. Record actual request schema, task status values, unit costs and latency.
5. Establish polling retention and safe polling interval; task expiry behaviour; provider rate limit; terminal-failure billing; result URL expiry; provider file/image retention and deletion conditions; terms on provider secondary use/training.
6. Confirm Inngest and Supabase plan/region/storage support; confirm Vercel Preview is available and can be protected.
7. Record privacy/data-transfer implications and write down accurate disclosures. If required provider terms are unknown, do not accept public participant images yet.
8. Confirm whether one YouCam account/key can be safely used across preview and production; otherwise create separate keys/projects.

**Files expected:** `docs/PHASE0_REPORT.md`, a redacted `docs/provider-fixtures/` sample (no API key, task secrets, real personal photos, or signed URLs), updated `docs/DECISION_LOG.md`.  
**Database:** None.  
**API:** No Groom APIs yet. Provider test scripts may be local, uncommitted or safe redacted.  
**UI:** None.  
**Tests:** Live provider proof-of-concept with test-consented images; verify both categories.  
**Acceptance:** All Phase 0 blockers in `ARCHITECTURE.md §17` have explicit PASS/FAIL/UNKNOWN evidence. Each endpoint and template ID is real and versioned. No unsupported provider assumption is hidden.  
**Verification:** Operator reviews report, test task IDs with redacted identifiers, results and credit usage.  
**Git:** Commit only documentation and safe test fixtures as `docs: record YouCam feasibility and provider contract`.  
**Stop gate:** If credentials/credit/templates or provider terms are unavailable, stop before invoking more than a minimal test. If API cannot work, do not move to a fake-only integration.

## Phase 1 — Repository/platform foundation + early skeletal frontend deployment

**Objective:** Give the operator a stable, navigable preview URL immediately, with every agreed route represented and all unfinished functions labelled.  
**Dependencies:** Phase 0 provides provider feasibility status; GitHub/Vercel access; agreed initial stack.  
**Tasks:**
1. Inspect existing repo (repeat if repo was changed after Phase 0), preserve valid work and confirm Next.js App Router / strict TypeScript / Node 22.
2. Set up Next.js 16.4.x, React compatible stable release, Tailwind 4, lint/typecheck/test scripts and lockfile.
3. Create shared page shell, header/footer and responsive layout with temporary neutral tokens.
4. Implement routes: `/`, `/sign-up`, `/sign-in`, `/dashboard`, `/dashboard/experiments/new`, `/dashboard/experiments/[id]`, `/dashboard/experiments/[id]/results`, `/p/[slug]`, `/docs`, `/privacy`, `/terms`, `/support`, not-found.
5. Add clear placeholder notices to screens not connected to backend; disable actions with explanation rather than fake success.
6. Add `/api/health` returning safe status only.
7. Configure Vercel Preview from GitHub, safe preview env and deployment protection if available. Ensure Preview uses test/sandbox data and no production API key/budget.
8. Deploy, verify all routes load and navigation works; record exact preview URL + commit SHA + deployment ID.

**Files/components:** `src/app/**`, `src/components/layout/**`, `src/components/ui/**`, `src/app/api/health/route.ts`, `src/lib/config/`, `.env.example`, `README.md`, `AI_HANDOFF.md`.  
**Database:** None.  
**API:** Health only.  
**UI:** All skeletal routes and global shell.  
**Tests:** Build, lint, typecheck, route smoke Playwright, mobile-width smoke and link check.  
**Acceptance:** Exact preview URL is live; navigation and route states work; visible placeholders are honest; no action falsely claims to save/generate; no secrets in bundle; correct deployed commit recorded.  
**Verification:** Operator opens preview on desktop and mobile and reviews route map.  
**Git:** `chore: establish Groom shell and preview deployment`.  
**Stop:** Provide preview URL and wait for acceptance before Phase 2.

## Phase 2 — Backend, database, auth, RLS and private storage foundation

**Objective:** Establish trusted data boundaries and owner authentication before business features.  
**Dependencies:** Phase 1 accepted; Supabase preview project established.  
**Tasks:**
1. Add Supabase migrations for all tables/indexes/constraints/triggers in `ARCHITECTURE.md §3`.
2. Enable RLS on all application tables; add owner-only policies; ensure sensitive participant/AI/response/usage/audit rows are never publicly readable.
3. Create private storage buckets `respondent-inputs` and `respondent-renders`; configure MIME/size limits and service/server access; no public listing/download.
4. Implement server/browser Supabase client adapters and auth middleware/layout guards.
5. Implement sign-up/sign-in/sign-out and idempotent profile provisioning.
6. Add health/readiness checks without leaking infrastructure information.
7. Add preview-only seeded owner and curated test template fixture clearly labelled non-production; provider templates must be real verified IDs before live studies.

**Files/components:** `supabase/migrations/**`, `src/lib/supabase/**`, `src/server/auth/**`, `src/app/(auth)/**`, `src/app/(owner)/**`, `src/types/database.ts`, `tests/db/**`.  
**Database:** All schema, RLS, storage config.  
**API:** Auth actions/route integration and health.  
**UI:** Sign-up/sign-in and protected dashboard shell connected.  
**Tests:** Owner auth E2E; profile idempotency; cross-owner RLS tests; private bucket signed URL tests; unauthenticated route guards.  
**Acceptance:** User A cannot read or mutate User B's experiments/records; service secrets not exposed; no public image object; migration works from clean DB.  
**Verification:** Run local Supabase tests and E2E against preview project.  
**Git:** `feat: add Supabase auth and secure data foundation`.  
**Checkpoint:** Update preview, commit SHA, changed flows, tests and known gaps in `AI_HANDOFF.md`; stop.

## Phase 3 — Core domain model, experiment lifecycle and owner APIs

**Objective:** Make the experiment creation and management domain real before image generation.  
**Dependencies:** Phase 2 accepted; curated template data available.  
**Tasks:**
1. Implement typed domain schemas and validators for experiment create/update and lifecycle.
2. Seed verified `style_templates` from actual Phase 0 catalogue; disable templates without a confirmed male label/visual review/asset rights.
3. Implement owner APIs for list/create/detail/update/publish/pause/resume/close/archive.
4. Implement DB transactions, row locking, ownership checks, active publication limit, slug generation and audit events.
5. Implement `GET /api/style-templates` and server-safe template mapping.
6. Build owner dashboard/experiment create/detail UI on temporary functional styles.

**Files/components:** `src/server/domain/experiments/**`, `src/server/repositories/**`, `src/app/api/experiments/**`, `src/app/api/style-templates/route.ts`, `src/app/(owner)/**`, `supabase/seed.sql`, `tests/domain/**`, `tests/api/experiments.test.ts`.  
**Database:** Template seed rows and lifecycle triggers/constraints.  
**API:** All owner experiment/template endpoints.  
**UI:** Dashboard, new experiment, experiment detail and lifecycle controls.  
**Tests:** Unit state-machine tests; schema validation tests; API ownership; duplicate template; slug collision; concurrent publish; immutability after publish; active experiment limit.  
**Acceptance:** Owner can create and publish valid 3–4 option study; category/options immutable after publish; no foreign owner access; all lifecycle rules match `PROJECT_SPEC.md`.  
**Verification:** API integration plus owner Playwright path on preview.  
**Git:** `feat: implement experiment lifecycle and template catalogue`.  
**Checkpoint:** Preview URL/SHA/tests/state report; stop.

## Phase 4 — Participant session, YouCam adapter and durable rendering

**Objective:** Complete the actual product's essential technical mechanism.  
**Dependencies:** Phase 0 live API passed; Phase 2 private storage; Phase 3 experiments and templates.  
**Tasks:**
1. Implement public-safe experiment read and anonymous session creation with consent/18+ affirmation.
2. Implement token generation/HMAC, cookie handling, session expiry and rate limits.
3. Implement signed Supabase upload URL and upload-complete verification; image normalisation with `sharp` and EXIF stripping.
4. Implement YouCam adapter using current confirmed contracts for File API, PUT binary bytes, hairstyle task, beard task, polling and result retrieval.
5. Implement Inngest event/function, bounded concurrency, polling, retries and unknown task creation handling.
6. Persist jobs and unit ledger with atomic budget reservation; implement task state transitions.
7. Store render outputs privately; issue short-lived participant-only signed URLs.
8. Implement scheduled cleanup and safe error mapping.

**Files/components:** `src/server/integrations/youcam/**`, `src/server/jobs/**`, `src/server/domain/participants/**`, `src/app/api/public/**`, `src/app/api/participant/**`, `src/app/api/inngest/route.ts`, `src/server/storage/**`, `tests/integrations/youcam/**`, `tests/jobs/**`.  
**Database:** Respondent sessions, AI jobs, budget and usage ledger; relevant policies/indexes.  
**API:** Public experiment, session, upload and participant state endpoints.  
**UI:** Participant intro, consent, image upload and processing states connected.  
**Tests:** Provider adapter mocked contract tests + one real test task per category; task retention/polling; signed PUT transfer; duplicate event; worker retry; ambiguous POST; unit-budget race; malicious file; path isolation; session expiry.  
**Acceptance:** Both study categories work with live provider test photo; actual result stored privately; no provider keys or signed provider URLs exposed; no duplicate task creation on retries; image constraints enforced; usage ledger correct.  
**Verification:** Operator watches full preview flow and checks job logs/ledger with redaction.  
**Git:** `feat: integrate YouCam grooming render pipeline`.  
**Checkpoint:** Update preview and `AI_HANDOFF.md`; stop.

## Phase 5 — Voting, aggregate results and complete MVP functionality

**Objective:** Complete the full participant and owner journeys before visual redesign.  
**Dependencies:** Phase 4 accepted.  
**Tasks:**
1. Implement stable randomised option order and comparison screen.
2. Implement rating 1–5 per option, exactly one favourite, client validation and server transaction.
3. Implement idempotent submit, counter update, response target closure and cleanup enqueue.
4. Implement owner aggregate results endpoint and UI; suppress results under 5; segment count suppression under 10.
5. Implement optional hair texture self-report only for hairstyle study; no AI inference.
6. Implement pause/closed/target-reached/expiry/failure recovery states.
7. Ensure owner sees no individual row/image/render/session.
8. Complete core privacy, terms, docs and support pages enough for testing.

**Files/components:** `src/server/domain/responses/**`, `src/app/api/participant/submit/route.ts`, `src/app/api/experiments/[id]/results/route.ts`, participant compare/vote screens, owner results screen, docs/privacy pages, tests.  
**Database:** Responses, ratings, aggregates/RPC, state/counter functions and indexes.  
**API:** Submit and aggregate results.  
**UI:** Comparison, rating, confirmation, aggregate report and all core failure states.  
**Tests:** Full transaction/idempotency tests; duplicate submit; response target concurrency; all aggregate formulas; threshold suppression; segment suppression; foreign owner tests; session cookie misuse tests.  
**Acceptance:** Full owner create→publish→share and participant consent→upload→render→compare→rate→submit→thank-you works end-to-end on deployed preview for hairstyle and beard experiments. Owner receives expected aggregate data only.  
**Verification:** Playwright E2E with isolated test data and actual YouCam outputs; record video/screenshare for evidence; no blocking failures.  
**Git:** `feat: complete Groom experiment and voting journeys`.  
**Hard gate:** **Do not begin final visual design until all critical MVP journeys pass E2E verification on preview and operator has seen evidence.** Stop for operator acceptance.

## Phase 6 — Reference-led design, mockups and explicit operator approval

**Objective:** Agree final visual design only after working behavior is proven.  
**Dependencies:** Phase 5 accepted.  
**Tasks:**
1. Ask operator for URLs/screenshots/images or written design direction. If none, present 2–3 reference candidates for selection; do not silently choose aesthetic.
2. Create mockups/prototypes for home, dashboard, experiment create, detail, public intro/consent, upload/processing, compare/vote, results and relevant error states.
3. Include at least desktop and mobile layouts; annotate design source influence and product-specific deviations.
4. Present mockups and gather feedback. Revise at least once if requested.
5. Record selected references, mockups, tokens, components and deviations in `docs/DESIGN_APPROVAL.md`.

**Files/components:** Mockup/prototype links or image files under `docs/design/`; `docs/DESIGN_APPROVAL.md`.  
**Database/API:** No changes.  
**UI:** Mockups only; application remains in temporary functional styling.  
**Tests:** Check mockups against route/permission/state inventory; no behavior changed.  
**Acceptance:** Operator explicitly replies with approval of the direction/mockups.  
**Git:** `docs: record approved Groom design direction` only after approval.  
**MANDATORY STOP:** The coding agent must not implement final visual styles before explicit operator approval. No approval means no Phase 7.

## Phase 7 — Implement approved final UI

**Objective:** Apply the approved aesthetic without changing product behavior or architecture.  
**Dependencies:** Explicit Phase 6 approval recorded.  
**Tasks:** Implement approved type/color/spacing/layout/component system; apply to functional screens; preserve forms, state transitions, API payloads, permissions and tests; update visual regression snapshots where useful.  
**Files/components:** `src/app/**`, `src/components/**`, `src/styles/**`, design tokens; `docs/DESIGN_APPROVAL.md`.  
**Database/API:** None expected. Any required contract change must be separately proposed and approved.  
**UI:** All screens and states under approved visual direction.  
**Tests:** Component/unit + Playwright; visual review on mobile/desktop; accessibility checks.  
**Acceptance:** Approved mockups matched; core E2E still passes; no API/business logic/architecture drift.  
**Git:** `feat: implement approved Groom visual design`.  
**Stop:** Provide preview URL/SHA/screens and await acceptance.

## Phase 8 — Complete frontend, accessibility, documentation and links

**Objective:** Finish the whole usable frontend, not just main screens.  
**Dependencies:** Phase 7 accepted.  
**Tasks:**
1. Verify all routes/nav/footer and responsive layouts at mobile/tablet/desktop widths.
2. Verify loading, empty, validation, error, success, permission-denied, unavailable, expired and not-found states.
3. Verify keyboard navigation, visible focus, labels, contrast, semantic elements, alt text and accessible feedback.
4. Complete title/description metadata, favicon/app icon, social metadata if appropriate, robots rules and not-found page.
5. Confirm `/docs`, `/privacy`, `/terms`, `/support` routes are real and accurate.
6. Add footer Documentation link to `/docs`; add GitHub link only to actual public repo URL in `GITHUB_REPOSITORY_URL`. If private or not approved for public exposure, document and obtain operator-approved omission.
7. Complete README and docs setup/configuration/testing/deployment/troubleshooting/operator/user guides.
8. Run link checker against deployed preview; remove all dead links, inert controls and unexplained placeholders.

**Files/components:** `README.md`, `docs/**`, `src/app/docs/**`, footer, metadata/icons, legal/support pages, `playwright` link/accessibility tests.  
**Database/API:** No new domain changes.  
**UI:** Full frontend completeness.  
**Tests:** Full E2E, links, responsive visual review, axe or equivalent accessibility checks.  
**Acceptance:** No dead or fake links; Docs destination real; GitHub link real or exception documented/approved; README/docs useful; all state screens and responsive/accessibility basics verified.  
**Git:** `docs: complete Groom operator documentation and frontend shell`.  
**Stop:** Provide final preview and link-check evidence; await production approval.

## Phase 9 — Security, regression, production approval and monitoring

**Objective:** Prove release readiness and deploy production only with operator authorisation.  
**Dependencies:** Phase 8 accepted; production accounts/configuration and privacy terms ready.  
**Tasks:**
1. Security review against `ARCHITECTURE.md §7`; adversarial owner/participant access tests.
2. Dependency/build audit, migration dry run, DB backup/restore check and secret scan.
3. Recheck YouCam unit budget, rate limits, polling retention, provider data terms and live account. Verify no test keys in production.
4. Verify cleanup/retention job and deletion evidence.
5. Run complete Playwright suite against production-like staging; verify webhook/job signature validation, logs redaction, no PII leakage.
6. Obtain explicit operator approval for release.
7. Deploy Vercel Production; smoke-test login, create/publish, participant test experiment, results and footer links using controlled test study.
8. Monitor logs, queue failures, API budget and storage; record release SHA/URL and rollback procedure.

**Files/components:** CI/security config, runbooks, `docs/RELEASE_CHECKLIST.md`, `docs/INCIDENT_RESPONSE.md`.  
**Database:** Apply reviewed migrations; backups verified.  
**API:** Production env.  
**UI:** Final production pages.  
**Tests:** Security suite, full E2E regression, build/lint/typecheck, smoke tests.  
**Acceptance:** All critical checks pass; no high severity vulnerability; provider contract is understood; production URL works; operator explicitly approves.  
**Git:** `release: Groom MVP v1.0.0`.  
**Stop:** Report deployed URL, commit, test evidence, open issues and monitoring status.

---

## Phase completion report template (mandatory at every checkpoint)

```text
Project: Groom
Current phase:
Assigned task:
Preview URL:
Deployed commit SHA / deployment ID:
Changes visible in preview:
Completed:
Incomplete / explicitly placeholder:
Tests run and exact results:
Manual verification performed:
Database/API changes:
Security/privacy notes:
Design reference/mockup status:
Operator approval status:
Documentation/footer-link status:
Known issues/blockers:
Git commit:
Next single task (do not start automatically):
```

## Schedule and go/no-go warning

As of 1 November 2026, the stated hackathon submission deadline is 2 November 2026 at 5:45 PM WAT. The full nine-phase sequence, live provider verification, privacy review, reliable MVP, demo and operator-controlled design gate cannot responsibly be guaranteed inside the remaining window. Do not bypass phases, fabricate API success, or skip the design approval gate to chase the deadline. Submit only if the working implementation and evidence actually meet the published rules. Otherwise preserve this blueprint as the basis for a proper post-hackathon build.
