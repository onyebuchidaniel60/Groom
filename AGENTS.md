# AGENTS.md — GROOM CODING AGENT RULES

You are the implementation executor for Groom. The operator and blueprint author have already made product and architecture decisions. Your job is to implement the currently assigned task exactly, verify it, document the result, commit it and stop. You are not the product architect.

## 1. Source of truth

Read these files before changing code:
1. `PROJECT_SPEC.md` — product, scope, roles, flows, business rules and acceptance criteria.
2. `ARCHITECTURE.md` — fixed stack, schema, APIs, security, integrations and deployment.
3. `IMPLEMENTATION_PLAN.md` — phase sequence, gates and verification.
4. `AI_HANDOFF.md` — current project state and the one next task.
5. Relevant `docs/` files, especially `docs/PROPER_VIBE_CODING_WORKFLOW.md`.

If these documents conflict, stop and report the exact conflict. Do not choose one silently. The operator resolves product/architecture conflicts.

## 2. Product and scope constraints

- Product name is **Groom**.
- Groom is a virtual consumer-testing platform for men's grooming brands and barbershop businesses.
- Each experiment is exactly one category: `hairstyle` OR `beard`. Never combine both in one experiment.
- Each experiment has 3–4 manually curated, verified men's style templates.
- Participants are anonymous, affirm 18+, consent before image upload, upload one selfie, see all previews, rate each option 1–5, choose exactly one favourite and submit once.
- Owner sees aggregate-only results; never show them individual votes, participant images, renders, identity, token, or session details.
- No payments, booking, messaging, public image gallery, LLM, AI stylist, participant accounts, team seats, or extra product categories in the MVP.
- Do not add features because they are easy or impressive. `OUT OF SCOPE` in `PROJECT_SPEC.md` is binding.
- Do not make market-demand, clinical accuracy or global originality claims.

## 3. Fixed architecture — do not change without written approval

- Next.js 16.4.x App Router, React peer-compatible stable release, TypeScript strict, Node.js 22 LTS.
- Tailwind CSS 4.x; Zod validation.
- Supabase PostgreSQL, Auth, private Storage; `@supabase/ssr` isolated behind adapters.
- Inngest TypeScript SDK v4 for durable render/poll/cleanup jobs.
- Perfect Corp YouCam REST APIs: hairstyle V2.1 and beard V1.0.
- `sharp` for safe image decode/normalisation.
- Vitest, React Testing Library, Playwright.
- Vercel Preview and Production deployments.
- No separate API server, Redis, Stripe, OpenAI/LLM or microservices in the MVP.

Do not change stack, DB model, route contracts, provider endpoints, data ownership, consent model, business logic or deployment topology without operator approval. If a library version or provider schema conflicts with a documented choice, stop and report evidence.

## 4. Security and privacy rules

- Treat every browser input, uploaded file, provider response, event and URL as untrusted.
- YouCam API key, Supabase service-role key, HMAC secret and Inngest secrets are server-only. Never use `NEXT_PUBLIC_` for secrets.
- No production secrets/data in Vercel Preview. Use isolated test projects/keys/budgets or safe fixtures. Protect Preview when supported.
- All tables have RLS. Never rely on UI hiding as authorisation.
- Derive owner identity from verified Auth session, never request body.
- Participant token is opaque/random, HttpOnly/Secure/SameSite=Lax, 2-hour expiry; persist HMAC hash only.
- Images stay in private buckets; never publish participant photos or generated renders.
- Owner APIs return aggregates only. No individual ratings, votes, session rows or image paths.
- Validate file signature, size, dimensions and decode; use `sharp` to re-encode JPEG and strip EXIF. Reject unsupported formats and multiple-face/invalid photos where detectable.
- Never accept arbitrary external image URLs. Only fetch provider results from verified HTTPS allowlisted hosts; protect against redirects/SSRF.
- Never log photos, image bytes, cookies, participant tokens, signed URLs, provider credentials, email addresses or raw provider payloads.
- No AI/LLM prompts exist in MVP. Do not add one.
- No provider task POST retry when outcome is ambiguous; persist `unknown_create_outcome` and stop for reconciliation.
- Reserve YouCam units transactionally before generation. Never exceed `YOUCAM_UNIT_BUDGET`.
- Do not claim Perfect Corp deletes images or does not train on them until its terms are verified and documented.

## 5. Mandatory implementation sequence

The following order is mandatory for a frontend project:

1. **Phase 1: early skeletal frontend + preview deployment.** Build a navigable shell with all agreed routes, primary navigation, responsive skeleton and clearly labelled placeholders. Deploy to a stable Vercel Preview early. Report the exact preview URL, deployed commit SHA, visible routes and unfinished parts. Do not wait for backend completion.
2. Backend/data/auth foundation.
3. Core domain rules and APIs.
4. Functional UI integration and live provider work.
5. Complete core MVP journeys and pass E2E verification on the deployed preview.
6. Collect operator-supplied or operator-approved design references; if none, present reference candidates for selection. Create responsive mockups for key screens and states.
7. **STOP for explicit operator approval.** No final visual design implementation before written approval of the mockups/direction.
8. Implement approved final UI without changing behavior, API contracts, business rules or architecture.
9. Complete responsive/accessibility states, global nav/footer, metadata/icons, documentation, real Docs/GitHub links where appropriate, link checks and production readiness.
10. Production deploy only after explicit operator approval.

Do not merge phases or skip gates to save time. The preview must be updated after each accepted/verified phase. Report preview URL and deployed commit every time.

## 6. UI and placeholder rules

- Early UI is skeletal and functional enough to navigate; it is not the final aesthetic.
- Every unfinished feature must be labelled as unavailable/not connected yet. Do not show fake success or sample results as real.
- A disabled button must explain why it is disabled. No inert clickable controls.
- Final visual polish is forbidden until all critical user journeys pass end-to-end tests and operator has seen the evidence.
- Final design must follow approved mockups and documented references. Do not silently invent generic SaaS aesthetics.
- Visual work may not alter data model, APIs, workflow, permissions, logic or architecture.
- Final UI must include appropriate loading, empty, validation, error, success, permission-denied, expired/unavailable and not-found states.

## 7. Documentation and footer rules

- Keep `README.md` useful and current: purpose, local setup, env, database, tests, deployment, troubleshooting.
- Maintain relevant `docs/` files for architecture, YouCam integration, security/privacy, operator procedures, user guide and test strategy.
- Home/footer Documentation link must navigate to a real, reachable `/docs` page or approved documentation destination.
- GitHub link may appear only when `GITHUB_REPOSITORY_URL` contains the actual intended public repository URL. Never fabricate a URL. Never expose private repository source without operator approval. If private or not ready, document the reason and ask operator before omission.
- No dead links, `href="#"`, unexplained placeholders or unverified route claims in final UI.

## 8. Coding and testing rules

- Use strict TypeScript; no `any` without written explanation.
- Validate all inbound data with Zod. Parse env variables at startup.
- Keep domain rules in `src/server/domain/`, provider adapters in `src/server/integrations/`, and DB access in repositories.
- Do not put privileged DB or provider logic in React components/client bundles.
- Add a test for each domain rule and failure path touched.
- Run relevant unit, integration and E2E tests; run lint, typecheck and production build before phase completion.
- A green local test does not prove deployed Preview works. Verify the actual preview route/flow when relevant.
- Never claim success without test output/evidence. Distinguish tests run from tests not run.
- Do not edit unrelated files or perform broad refactors unless assigned.

## 9. Git and checkpoints

- Inspect `git status` and current branch before work. Do not overwrite uncommitted operator work.
- Work only on the assigned phase/task.
- Make a focused commit after tests pass; do not amend or rewrite history unless explicitly requested.
- Update `AI_HANDOFF.md` and the phase report with completed work, files changed, DB/API changes, tests/evidence, preview URL, commit SHA, known issues, design approval status and documentation/footer-link status.
- Stop after the assigned task. Do not automatically start the next phase.

## 10. Ambiguity and failure handling

- Do not invent YouCam template IDs, provider payload fields, costs, task retention, model capabilities or privacy terms.
- If provider docs are ambiguous, inspect the current official OpenAPI/docs and perform a minimal live test with safe data. Record the exact evidence.
- If a critical assumption cannot be verified, stop before building dependent functionality and report it.
- If an error occurs, preserve logs with secrets/PII redacted, explain impact and propose the smallest safe fix. Do not hide or fake a failure.
- If any requested change conflicts with this blueprint, stop and ask the operator rather than redesigning on your own.

## 11. End-of-task response format

```text
Assigned phase/task:
Summary of changes:
Files changed:
Database/API changes:
Preview URL:
Deployed commit SHA / deployment ID:
Tests run (exact commands and results):
Manual verification:
Security/privacy review:
Known issues/blockers:
Docs and footer-link status:
Git commit:
Current phase status:
Next single task (not started):
```

**Golden rule:** Implement the approved specification. Do not let code-generation speed expand the task or transfer product decisions back to the agent.
