# Groom — Incident Response Runbook (Initial)

## Severity examples
- **Critical:** participant images publicly exposed; provider/API secret compromised; cross-owner data access; unauthorised image processing; widespread DB disclosure.
- **High:** repeated task duplication/billing, cleanup backlog exposing retained images beyond policy, authentication bypass, sustained provider misuse.
- **Medium:** render worker failures, budget unexpectedly depleted, partial outage with no data exposure.

## Immediate containment
1. Set `GROOM_NEW_SESSIONS_ENABLED=false`.
2. If rendering is unsafe or provider key compromised, set `GROOM_RENDER_ENABLED=false` and revoke/rotate YouCam key.
3. Pause affected experiments.
4. If participant token HMAC secret is compromised, rotate secret and invalidate participant sessions.
5. If Supabase service-role key is compromised, rotate it immediately; review audit/log access and storage policies.
6. Preserve redacted logs, job IDs and timestamps; do not copy participant images into chat or tickets.

## Investigation
- Determine affected systems, data categories, time range and number of participant sessions.
- Verify whether source images/renders, participant tokens, owner accounts, ratings or API credentials were accessed.
- Identify whether vendor/hosting providers need incident escalation.
- Reconcile usage ledger and provider task list if task duplication suspected.
- Verify cleanup jobs and private storage ACL after fix.

## Recovery
- Patch and test in isolated Preview.
- Run access-control and storage penetration tests.
- Rotate/invalidate credentials and sessions as appropriate.
- Resume only after operator approval.

## Notification and recordkeeping
The operator/privacy lead obtains legal advice on applicable notification duties and timelines. Record incident timeline, containment, scope, root cause, remediation and follow-up actions. This runbook does not determine statutory notification obligations.
