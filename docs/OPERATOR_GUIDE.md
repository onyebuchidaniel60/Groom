# Groom — Operator Guide (Pre-Implementation)

This guide describes intended operation. Application screens/commands do not exist yet; the coding agent must complete exact commands and URLs after implementation.

## 1. Before any public participant study

1. Confirm Groom privacy notice and terms have been reviewed.
2. Confirm YouCam API account is active and units redeemed; verify current remaining balance in official console.
3. Complete a live hairstyle and beard API test with consented test images.
4. Verify provider template IDs and manually curate men's styles.
5. Confirm provider's input/output retention, data processing, cross-border and secondary-use terms.
6. Configure Preview with isolated Supabase project, test storage and low unit budget.
7. Verify `GROOM_NEW_SESSIONS_ENABLED` and `GROOM_RENDER_ENABLED` kill switches.
8. Confirm cleanup workers are running and tested.

## 2. Template curation

- Fetch real template IDs from official YouCam template endpoints.
- Inspect actual rendered output and intended gender/style fit; API's general support for men does not prove every template is a men's style.
- Confirm preview thumbnail display rights.
- Write a human-readable name and a factual description.
- Seed only approved templates; set `men_verified=true` after review.
- Mark templates inactive if provider removes them, output quality changes, or rights are unclear.

## 3. Unit budget

- Confirm account balance from YouCam console.
- Set internal app budget below account balance, retaining test/incident reserve.
- Default budget is 700 only if at least 1,000 units are actually available; otherwise lower it.
- Current docs list 2 units per hairstyle V2.1 preset or beard V1.0 job; recheck before changing constants.
- A four-option participant costs an estimated 8 units. A 20-person experiment is an estimated 160 units before retries/tests.
- Monitor `usage_ledger` totals; reconcile `unknown` rows in provider account before releasing reserved budget.
- Do not manually reset ledger counters without a documented reconciliation.

## 4. Running studies

1. Owner creates draft and selects one category.
2. Owner selects 3–4 verified templates and defines response target.
3. Owner reviews public text for accuracy and no misleading promises.
4. Owner publishes and shares the public link.
5. Owner can pause, resume (if not closed), close or archive.
6. Results unlock after 5 completed responses. Subgroup results appear only at N≥10.
7. Explain to customers that small-panel results are directional, not statistically representative.

## 5. Monitoring

Monitor:
- Inngest job run state, task latency and retries.
- `unknown_create_outcome` count.
- Failed provider jobs by safe error code.
- reserved/committed/released/unknown units.
- active participant sessions and expirations.
- cleanup backlog and private storage usage.
- Vercel errors, Supabase DB/storage status and auth errors.

Do not use logs to inspect participant faces or identify participants. Escalate repeated provider errors instead of retrying task POST blindly.

## 6. Incident response

- Stop new starts with `GROOM_NEW_SESSIONS_ENABLED=false`.
- Stop new image processing with `GROOM_RENDER_ENABLED=false`.
- Pause affected study, preserve redacted event IDs and timestamps, rotate compromised secrets.
- Revoke active sessions if participant HMAC secret is exposed.
- Investigate storage permissions, signed URLs and DB access logs.
- Follow the incident-response and legal assessment process before notifying individuals or authorities.

## 7. Releasing updates

- Implement one assigned phase/task, test, review, fix and commit.
- Update the stable Preview after each verified phase.
- Record preview URL, SHA, changes and test evidence in `AI_HANDOFF.md`.
- Do not implement final visual UI until core E2E gate passes and operator explicitly approves mockups.
- Deploy Production only with explicit operator approval.
