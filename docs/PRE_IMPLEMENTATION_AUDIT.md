# Groom — Pre-Implementation Audit

## 1. Contradictions / design decisions resolved

- **Experiment category:** one experiment is hairstyle OR beard; not both.
- **Participant access:** one active anonymous browser session at a time; one response per issued session. Groom does not claim one person/one vote because identity is not verified.
- **Cookie path:** participant token cookie is sent to `/api` routes; it is HttpOnly/Secure/SameSite=Lax and expires after two hours. It is not scoped to the public `/p/` page because the API endpoints must receive it.
- **Image path:** browser upload goes to a private Supabase bucket; server validates/normalises; provider File API returns its own signed PUT URL, to which the server transfers actual bytes. Only then is a task created.
- **Face/pose validation:** Groom performs file/dimension/pixel validation; YouCam performs task-specific face/pose/landmark validation. No separate face-recognition or third-party face-detection model in MVP.
- **Result privacy:** owners see only aggregate data. Owner-specific aggregation function checks `auth.uid()` internally and returns grouped values only.
- **Render failure:** no partial comparison is shown; ambiguous provider task create is not blindly retried.
- **Cost:** 2 units per selected current preset operation is an estimate from current docs, not a guarantee about user's account. Budget is checked transactionally.
- **Design:** UI structure is planned now; final aesthetic is unapproved and cannot be implemented until mockups are presented and explicitly approved.

## 2. Remaining unresolved items (hard Phase 0 blockers)

1. Does the operator have a current YouCam API key and redeemed balance?
2. Which returned hairstyle templates are demonstrably appropriate men's styles, and may preview assets be displayed?
3. Which beard presets are suitable and what are their actual provider IDs?
4. What exact task polling retention/interval and timeouts apply to current account/API version?
5. What is provider billing for task creation timeout, failed task and task retention expiry?
6. What are Perfect Corp's data-processing, retention, deletion, region, subprocessor and secondary-use terms?
7. What image/result URL retention applies after successful task?
8. Does the current project have a GitHub repository, Supabase and Vercel project already?
9. What Supabase region/plan and Vercel Preview protection are available?
10. Can Inngest v4 execute the required task schedule under the chosen plan?

## 3. Scope risks

- A complete, polished version is unlikely to be responsibly built before the 2 November 2026 5:45 PM WAT hackathon deadline.
- Novelty of the exact Groom workflow remains NOT READY; further product/market and hackathon prior-art research could require redesign.
- 18+ is self-attestation, not verified age.
- Anonymous sessions cannot guarantee one real person one vote.
- Hair texture segments are self-reported and small panel results are directional.
- YouCam output is simulation, not guaranteed barber result.

## 4. No-code audit result

No application code has been created in this blueprint. The package contains only specification, architecture, implementation plan, agent instructions and project documentation. The coding agent's first task is read-only repository/account inspection.

## 5. Advancement conditions

Do not start live photo processing until API credentials, template IDs, live tasks, cost, polling retention and vendor privacy terms are verified. Do not start final UI visual design before E2E completion and mockup approval. Do not deploy production without operator approval.
