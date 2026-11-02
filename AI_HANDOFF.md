# AI_HANDOFF.md — GROOM PROJECT STATE

**Project:** Groom  
**Blueprint version:** 1.0  
**Handoff state:** Pre-implementation; no application code has been written.  
**Current phase:** Phase 0 — repository, account and critical integration reconnaissance.  
**Current preview URL:** Not deployed yet. Do not invent or reuse a URL.  
**Deployed commit:** None.  
**Design references/mockups:** Not collected; no final design direction approved.  
**Documentation/footer status:** Blueprint docs drafted. In-app `/docs` and repository links not implemented. Actual GitHub URL not yet known.

## Product snapshot

Groom is a virtual consumer-testing platform for men's grooming brands and barbershop businesses to test proposed hairstyle OR beard-style collections on consenting adults before launch. One experiment uses only one category, 3–4 curated verified men's templates, one anonymous photo-based participant session, one render per option, a 1–5 rating per option and exactly one first-choice vote. Owners see aggregate-only results.

## Fixed stack

Next.js 16.4.x App Router / TypeScript strict / React compatible stable release / Tailwind CSS 4.x / Zod / Supabase PostgreSQL + Auth + private Storage / Inngest v4 / Perfect Corp YouCam REST / `sharp` / Vitest / React Testing Library / Playwright / Vercel.

## Critical product/security rules

- Experiment type is exactly one of `hairstyle` or `beard`; never combine.
- 3–4 valid, manually curated men's style templates.
- Participants are anonymous, 18+ affirmation and affirmative consent required before upload.
- One JPG/JPEG selfie; private storage; no raw images shown to owners.
- One complete preview for each option; one rating (1–5) per option; exactly one favourite; one response per session.
- Owners receive aggregates only; no individual votes, images, renders or session rows.
- Results unlock at 5 completed responses; subgroup display requires N≥10.
- No payments, booking, LLM, recommendations, public gallery, participant accounts or expanded categories.
- Use transactional API-unit reservations; global internal budget defaults to 700 only after verifying actual credit. Cost currently documented as 2 units per hairstyle V2.1 preset or beard V1.0 task; re-check live.
- Do not retry ambiguous YouCam task-creation POST; mark unknown and reconcile.
- Preview uses safe test configuration/data; no production keys or participant images.

## Phase 0 verification still required

1. Confirm GitHub repository and inspect it without overwriting existing work.
2. Confirm real YouCam API key and redeemed balance.
3. Fetch actual hairstyle and beard template lists; curate male templates with real IDs and valid preview/usage rights.
4. Run one real hairstyle V2.1 and beard V1.0 test through File API → signed PUT byte upload → task POST → poll → result download.
5. Verify task retention/poll interval, API rate limits, billing on failed/expired jobs, result URL lifetime and data retention/secondary-use terms.
6. Verify Supabase project/region/storage and Inngest/Vercel environments.
7. Review privacy notice with qualified counsel before commercial deployment or broad collection.

## Current test evidence

No Groom application tests have been run. Blueprint research checked current official provider docs but did not establish that the user's account has a working key or redeemed units. No live API task is claimed to have succeeded for this project.

## Preview / deployment record

- URL: None.
- Commit SHA: None.
- Deployed changes: None.
- Incomplete areas: All application routes and functionality.

## Design approval record

- References: Not collected.
- Mockups: Not created.
- Operator approval: **NOT GRANTED**.
- Rule: Final UI visual implementation must stop until mockups are presented and operator explicitly approves.

## Docs / footer record

- Core blueprint documents: `PROJECT_SPEC.md`, `ARCHITECTURE.md`, `IMPLEMENTATION_PLAN.md`, `AGENTS.md`, `AI_HANDOFF.md`.
- README and supplemental operator docs are included in this package.
- In-app `/docs` route is planned, not implemented.
- Actual GitHub URL: unknown. Do not create a fake footer destination. If repository will be public, configure actual URL after creation; if private, omit only with documented operator approval.

## Single next task

**Phase 0, task 1 only:** Inspect the actual GitHub repository and connected deployment/account state without modifying any files. Report repository contents, current branch/working tree, runtime/dependencies, existing app entry points, deployment setup, and the exact provider/account information still missing. Then stop and request operator review.
