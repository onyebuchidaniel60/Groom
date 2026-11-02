# Groom — Decision Log

| ID | Decision | Status | Reason / evidence | Reopen condition |
|---|---|---|---|---|
| D-001 | Project name is Groom. | Selected | Operator explicitly chose the name. | Operator changes name. |
| D-002 | Product is pre-launch men's grooming preference research, not a booking app. | Proposed/selected direction | Distinct workflow focus and owner intent. | Product discovery changes the problem. |
| D-003 | One experiment is either hairstyle or beard, never combined. | Fixed in blueprint | Keeps comparisons coherent and avoids chained model complexity. | Operator approves future expansion. |
| D-004 | Each experiment has 3–4 curated men's templates. | Proposed MVP rule | Small option set and bounded API cost. | Usability test demonstrates need to change. |
| D-005 | Participants are anonymous and 18+. | Fixed | Data minimisation and short participant journey. | Abuse/legal review requires adjustment. |
| D-006 | Owner sees aggregate-only results. | Fixed | Privacy boundary; avoids individual vote/photo disclosure. | Separate reviewed research model approved. |
| D-007 | Results unlock at 5 responses; segments require 10. | Proposed threshold | Avoid displaying unstable/small-group information; not a statistical guarantee. | User/research/privacy validation. |
| D-008 | Next.js modular monolith, Supabase, Inngest, YouCam, Vercel. | Fixed pending Phase 0 access verification | Minimal stack that covers routes, DB, auth, storage, jobs and deploy. | Verified blocker with operator approval. |
| D-009 | No payments, LLM, chat, booking or participant accounts in MVP. | Fixed | Not required to prove core proposition. | Post-MVP evidence and approval. |
| D-010 | Final visual design waits until full functionality E2E passes and mockups receive explicit approval. | Non-negotiable workflow gate | Operator methodology. | Never waive silently. |
| D-011 | GitHub footer link only when actual public URL exists; docs route must be real. | Non-negotiable | Prevent fabricated links/private source exposure. | Repository visibility decided. |
| D-012 | Novelty status remains NOT READY. | Fixed evidence status | Strong adjacent competitors; full 3-pass research incomplete. | Three documented prior-art passes find no close public equivalent. |
| D-013 | Hackathon deadline does not justify skipping blueprint/QA/approval sequence. | Fixed | Deadline is too close for a responsible guarantee. | Operator chooses a knowingly reduced submission scope; must still comply with rules. |
