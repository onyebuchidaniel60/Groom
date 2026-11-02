# Groom — YouCam API Integration Notes

**Status:** Design contract based on current official documentation; live account verification remains Phase 0.  
**Official references (accessed 1 November 2026):**
- Hairstyle V2.1: https://docs.perfectcorp.com/reference/ai_hairstyle/section/overview/integration-guide
- Beard style: https://docs.perfectcorp.com/reference/ai_beard_style
- Quick start / file upload: https://docs.perfectcorp.com/develop/quick_start_guide
- Release notes: https://docs.perfectcorp.com/release/changelog

## 1. Groom scope

Groom uses only two YouCam capabilities in the MVP:

- **AI Hairstyle Virtual Try-On V2.1 preset mode** for hairstyle studies.
- **AI Beard Style Generator V1.0** for beard-style studies.

One experiment uses one category. Never chain both APIs for one participant in the MVP. The product stores provider template IDs in the curated `style_templates` catalogue; no template ID is hard-coded from memory or invented.

## 2. Documented endpoints and current unit costs

| Operation | Endpoint | Current published units |
|---|---|---:|
| Hairstyle templates | `GET /s2s/v2.1/task/template/hair-transfer` | 0 assumed; verify account usage |
| Hairstyle preset task | `POST /s2s/v2.1/task/hair-transfer` | 2 per task (V2.1 preset) |
| Hairstyle task status | `GET /s2s/v2.1/task/hair-transfer/{task_id}` | 0 assumed; verify account usage |
| Beard templates | `GET /s2s/v2.0/task/template/beard-style` | 0 assumed; verify account usage |
| Beard task | `POST /s2s/v2.0/task/beard-style` | 2 per task |
| Beard task status | `GET /s2s/v2.0/task/beard-style/{task_id}` | 0 while running according to docs; account billing still must be checked |
| File API | `POST /s2s/v2.0/file` | No separate unit assumed; verify |

The published docs currently describe hairstyle V2.1 preset mode as 2 units and Beard V1.0 as 2 units. Unit consumption and billing terms can change; confirm in the user's live API account before public use.

A completed participant requires 3 or 4 transformations: estimated 6 or 8 units. A 20-response, 4-option study estimates 160 task units, excluding test calls and any provider billing for unusual failures.

## 3. Mandatory File API sequence

The provider docs explicitly warn that a File API call alone does not transfer the image bytes. The returned signed upload URL must receive an HTTP PUT containing the actual file bytes.

1. Server calls `POST /s2s/v2.0/file` with the documented file metadata.
2. Parse the returned `file_id` and `requests.url` with Zod.
3. Server PUTs the normalised JPEG bytes to that URL using the returned/required content type and length.
4. Require a successful HTTP status from the PUT.
5. Only then call hairstyle/beard task endpoint using `src_file_id`.
6. Persist `file_id` and task ID in the AI job. Do not persist the signed upload URL.

If upload PUT fails, do not create a task. If upload succeeds but task creation has ambiguous network outcome, mark `unknown_create_outcome`; do not blindly create another task.

## 4. Hairstyle V2.1

**Documented endpoint:** `POST /s2s/v2.1/task/hair-transfer`  
**Template list:** `GET /s2s/v2.1/task/template/hair-transfer`  
**Status:** `GET /s2s/v2.1/task/hair-transfer/{task_id}`

The integration guide describes `template_id` for predefined styles and `src_file_id` for the participant source file. Use the current official OpenAPI description for exact body structure and required fields. Do not assume optional parameters from V2.0 apply to V2.1.

Documented input constraints in the current guide:
- JPG/JPEG.
- File under 10 MB.
- Long side <=1024 px.
- Face width >=128 px.
- Single full face; full hair visible and shoulders visible for the hairstyle operation.
- Pose ranges documented by provider: pitch -10° to +10°, yaw -45° to +45°, roll -15° to +15°.
- Provider may reject unsupported landmarks, hair length or face pose.

The guide says the capability supports both women and men. Groom must still manually inspect and curate male-presenting style templates; capability support does not establish that every template is suitable for men's grooming.

## 5. Beard Style V1.0

**Documented endpoint:** `POST /s2s/v2.0/task/beard-style`  
**Template list:** `GET /s2s/v2.0/task/template/beard-style`  
**Status:** `GET /s2s/v2.0/task/beard-style/{task_id}`

The provider describes beard simulations including trim beard, stubble, full beard, circle beard, moustache, goatee and other presets. Use actual template IDs returned by the official endpoint.

Documented input constraints:
- JPG/JPEG.
- File under 10 MB.
- Long side <1024 px.
- Face width >256 px.
- Single full face visible.
- Yaw within -30° to +30°.

The provider specifically warns that the task must be polled within its retention period. If it is not polled within that period, the task can time out even if processing was successful and units may still be consumed. Phase 0 must discover the actual safe polling interval/retention window for the account and implement polling well within it.

## 6. Template catalogue

At Phase 0:
1. Fetch current lists from both endpoints using the actual account key.
2. Store a redacted JSON snapshot in private operator docs; remove API keys, signed URLs and any participant information.
3. Inspect each candidate's preview/template image manually.
4. Select a small coherent set of male grooming styles for demo use.
5. Confirm usage rights for any preview thumbnail displayed in Groom.
6. Seed only approved rows in `style_templates`; mark `men_verified=true` only after manual review.
7. Do not assume provider returns a gender field; manually curate if metadata is inconclusive.

If template preview assets cannot legally/technically be displayed, use a permitted internally created text-only option or another operator-approved asset. Do not hotlink an unapproved provider URL or invent a thumbnail.

## 7. Asynchronous job pattern

All API calls happen server-side. Long-running polling never holds open a browser request or Next.js route handler.

```text
Participant uploads private image
    -> Groom verifies/normalises JPEG
    -> Groom File API request
    -> Groom PUTs bytes to signed provider URL
    -> Groom POSTs VTO task
    -> Groom persists task_id immediately
    -> Inngest sleeps/polls until terminal state
    -> Groom downloads result from approved provider host
    -> Groom stores output in private bucket
    -> Participant session becomes ready when every option succeeded
```

The UI polls Groom's own session-status API, not Perfect Corp. Return coarse progress states such as `uploading`, `rendering`, `ready`, `failed`; never return provider task IDs or raw URLs.

## 8. Failure and retry rules

| Condition | Action |
|---|---|
| File invalid before provider call | Reject; participant can re-upload once. No task is created. |
| Provider file upload fails | Retry the PUT only if provider confirms upload URL remains valid; otherwise create a new file record safely. No AI task yet. |
| Task POST receives clear validation error | Mark failed; do not retry same invalid input. |
| Task POST times out / connection drops before task ID | Mark `unknown_create_outcome`; no automatic POST retry. Operator checks account/tasks/usage. |
| Task status GET transient 5xx/network | Bounded retry with jitter inside retention window. |
| Task status becomes error | Mark terminal failed; reconcile unit according to provider's current billing rule. |
| Task never reaches terminal before deadline | Mark failed/unknown and stop polling before retention expires; operator reconciliation. |
| Result URL fetch fails | Retry short bounded downloads; do not expose URL to browser or logs. |
| Supabase result upload fails | Retry result storage while keeping provider result available; alert if provider URL may expire. |
| One of several options fails | Do not show partial comparison; fail session and use controlled recovery, not automatic full duplicate run. |

## 9. Result URL security

- Result URL is an external signed URL and must be treated as a credential.
- Download server-side immediately; enforce HTTPS, exact allowlisted host, redirect validation, timeouts and maximum response size.
- Do not return or persist that URL as a durable database value.
- Store downloaded output in private Supabase bucket, serve a short-lived signed URL only to the matching participant token.
- Delete render objects upon submission or expiry.

## 10. Live provider verification checklist

- [ ] API key is active and stored server-side.
- [ ] Actual credited balance and internal budget are confirmed.
- [ ] Current OpenAPI schemas saved/inspected for the exact endpoint versions.
- [ ] Male hairstyles and beard templates fetched and manually curated.
- [ ] One hairstyle task completes with an approved adult test image.
- [ ] One beard task completes with an approved adult test image.
- [ ] File PUT step is verified with actual byte transfer.
- [ ] Poll interval and task retention are established.
- [ ] Unit cost and terminal failure billing are checked.
- [ ] Output URL lifetime and output format are checked.
- [ ] Provider retention, deletion and secondary-use terms are reviewed.
- [ ] No key, signed URL, real participant image or personal identifier is logged.
