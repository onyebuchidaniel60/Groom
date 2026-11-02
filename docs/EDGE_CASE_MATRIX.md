# Groom — Error and Edge-Case Matrix

| Area | Condition | Expected behaviour | Retry/recovery | Must test |
|---|---|---|---|---|
| Auth | Invalid credentials | Generic sign-in error; no email enumeration | User retries | Yes |
| Auth | Auth succeeds but profile insert fails | Dashboard remains unavailable; profile creation retries idempotently | Retry provisioning | Yes |
| Owner access | Owner guesses another experiment UUID | 404; no details leaked | None | Yes |
| Template | Template API unavailable | Disable create/publish; show catalogue unavailable | Owner retries later | Yes |
| Template | Template deactivated after draft | Publish blocked with option-specific error; draft preserved | Owner replaces template | Yes |
| Experiment | 2 or fewer / 5 or more options | Validation error | Correct form | Yes |
| Experiment | Mixed hairstyle and beard options | Reject | Rebuild draft | Yes |
| Experiment | Publish clicked twice | Idempotent response; one status change/audit event | Return existing state | Yes |
| Experiment | Pause while new participant starts | Experiment row lock decides order; a start only succeeds if it reserved capacity while published | Participant gets unavailable if pause wins | Yes |
| Experiment | Target reached by concurrent submissions | Atomic count closes study once; no extra session | Show closed state | Yes |
| Session | Missing consent | No session, no storage upload, no provider call | User may affirm consent | Yes |
| Session | Under-18 answer | Block flow; do not create participant session | None | Yes |
| Session | Cookie disabled | Explain session cookie is required; do not silently downgrade | Enable cookies/restart | Yes |
| Session | Expired cookie/session | 410/session expired; no render or submit | Restart only if study still open | Yes |
| Session | Token guessed or changed | Deny; constant-time hash comparison | None | Yes |
| Upload | Extension says JPEG but bytes are PNG/HTML | Reject on signature/decode | Re-upload valid JPG | Yes |
| Upload | >10MB | Reject before decode/provider | Compress and retry | Yes |
| Upload | JPEG decompression bomb/huge dimensions | Reject based on pixel cap/time | New image | Yes |
| Upload | More than one face / no face | Reject where detector/Provider detects; user instructions say one face | User re-uploads | Yes |
| Upload | Hair not visible / shoulders absent | Explain hairstyle photo guidance | Re-upload | Yes |
| Upload | Beard face too small / yaw excessive | Explain stricter beard photo guidance | Re-upload | Yes |
| Upload | Upload URL expires | Reject completion; issue new signed URL if session valid | Retry upload | Yes |
| Upload | Storage receives object but completion callback lost | Idempotently verify same session-owned object and continue | Retry completion | Yes |
| Upload | Arbitrary external source URL supplied | Reject; app only accepts signed upload path | None | Yes |
| Provider file | File API 401/403 | Mark integration unavailable; do not expose raw error | Operator fixes key | Yes |
| Provider file | Signed PUT fails | Do not call VTO task | Retry PUT only if URL valid; otherwise new provider file | Yes |
| Provider task | Provider validation error | Mark job failed with safe code | Participant may restart after controlled recovery | Yes |
| Provider task | POST timeout before task ID | Mark `unknown_create_outcome` | Operator reconciliation; never automatic POST retry | Yes |
| Provider task | Duplicate Inngest event | Reuse existing job/task state | No new provider task | Yes |
| Provider task | Poll transient 5xx | Bounded retry within retention window | Inngest retry/sleep | Yes |
| Provider task | Poll delayed beyond retention | Do not assume task still exists or is free | Mark unknown/failed and reconcile | Yes |
| Provider task | Unknown task status value | Fail closed; log sanitized code | Operator review | Yes |
| Provider result | Result URL host not allowlisted | Reject download | Operator investigates | Yes |
| Provider result | Redirect to private/metadata IP | Block | None | Yes |
| Provider result | Downloaded output too large/invalid image | Reject and mark safe failure | Bounded retry only if provider output may be transient | Yes |
| Rendering | One of 3–4 options fails | Do not show partial comparison; session enters failed/recovery path | One controlled full retry only after checking outstanding tasks/budget | Yes |
| Budget | Internal unit budget exhausted | Block new session/render; show temporary unavailable | Operator reconciles/adds budget | Yes |
| Budget | Billing outcome unknown | Units remain unknown/reserved | Manual reconciliation | Yes |
| Vote | Missing rating | Submit rejected; highlight missing option | Participant fills field | Yes |
| Vote | Rating outside 1–5 | Reject server-side | Correct selection | Yes |
| Vote | No / multiple favourites | Reject server-side | Select exactly one | Yes |
| Vote | Duplicate submit | Return existing thank-you state; do not increment count twice | Idempotent | Yes |
| Vote | Submit races with close/target | Transaction determines acceptance; response target never exceeded | Return accepted or closed | Yes |
| Results | <5 completed responses | Hide aggregates, show unlock progress | Wait for more valid responses | Yes |
| Results | Segment <10 | Suppress segment value/count detail | Wait for threshold | Yes |
| Results | Owner tries direct raw response endpoint | No endpoint; 404/403 | None | Yes |
| Cleanup | Storage deletion transient error | Record failed deletion and retry | Scheduled cleanup | Yes |
| Cleanup | Database row deleted but storage object remains | Reconciliation job finds orphan by prefix/age and removes it | Retry | Yes |
| Network | Participant refreshes processing page | Rehydrate session from cookie and show current status | Continue | Yes |
| Network | Browser disconnects during submit | Idempotency check resolves state on reconnect | Retry same request key | Yes |
| Deployment | Preview env missing provider key | Show disabled test mode; no secrets or false success | Operator configures safe test key | Yes |
| Deployment | Production env points to Preview DB or vice versa | Startup configuration validation/health alert; block unsafe writes | Correct config | Yes |

## Recovery principles

- Fail closed for auth, ownership, consent, storage privacy and budget.
- Never present partial option renders as a fair comparison.
- Never retry an ambiguous task-create POST automatically.
- Do not throw away valid participant progress when a safe idempotent retry is possible.
- Cleanup and ledger reconciliation must be idempotent.
- Expose a human-readable explanation to users and a safe internal code to operators.
