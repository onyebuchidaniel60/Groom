# Groom — Security and Privacy Requirements

## 1. Scope and data controller

Groom processes personal photographs uploaded by adult participants for the limited purpose of virtual hairstyle or beard-style previews and collection-preference research. The business/brand owner receives aggregated research outputs only. Groom's operator determines product purposes and is responsible for the privacy notice, lawful basis, data-subject requests, processor due diligence and incident response.

This document is an engineering control plan, not legal advice. Obtain qualified privacy/legal review before a broad public or commercial launch. Nigeria's Data Protection Act 2023 is a relevant primary law for processing in Nigeria; other jurisdictions may impose additional requirements.

Official source: Nigeria Data Protection Act 2023, NDPC-hosted copy: https://ndpc.gov.ng/wp-content/uploads/2024/03/Nigeria_Data_Protection_Act_2023.pdf

## 2. Data minimisation

- Do not request participant name, email, phone, account or social identity.
- No raw IP address persisted in Groom database.
- Request only one participant photo and necessary consent/session data.
- Hair texture is optional self-report; no image-based inference of ethnicity, race, gender identity, age or health.
- No participant profile, public gallery, comments or marketing tracking.
- Do not store full provider request/response blobs; persist required identifiers/status only.

## 3. Consent experience

Before upload, show clear text describing:
- The photo will be processed by Groom and transmitted to Perfect Corp/YouCam to create the requested virtual grooming previews.
- The purpose is comparing proposed grooming styles in this experiment.
- The participant must be at least 18.
- The experiment owner receives aggregate preferences, not the participant's photo or individual vote.
- Groom stores the photo/results temporarily for processing and deletes them according to the stated retention schedule.
- The participant can stop before submitting and how to contact Groom about a privacy request.
- Provider processing is subject to Perfect Corp's applicable terms and privacy policy; link to current official policy after verifying it.

Consent checkbox must be unchecked by default; no consent inferred from continuing or silence. Store consent text/version and timestamp. Age affirmation is a separate explicit checkbox. If a participant declines, do not create an upload session or transfer an image.

Do not claim that a checkbox alone resolves every legal obligation. Confirm controller/processor roles, lawful basis, cross-border transfer terms, retention and rights handling before commercial release.

## 4. Retention/deletion target

| Data | Groom target | Trigger |
|---|---|---|
| Raw source image | Delete as soon as all required provider jobs are terminal and input is no longer needed; target within 24 hours maximum | Render completion / failure cleanup |
| Generated renders | Keep only until participant submits or session expires; delete immediately after submit or at expiry | Submit / TTL |
| Participant cookie | 2 hours maximum | Session expiry |
| Token hash/session audit | Keep minimum record required to prevent duplicate submissions and support security; define final legal retention before launch | Retention schedule |
| Aggregated result rows | Retain for owner study history unless deletion request/retention policy requires removal | Owner archive/deletion process |
| Provider-side files/tasks | Unknown until provider terms/account settings verified | Do not claim deletion without proof |

Scheduled cleanup runs at least hourly. Every deletion attempt is idempotent and logged without PII. Failed cleanup retries and alerts the operator. A participant image is never kept indefinitely just because a background worker failed.

## 5. Storage and access controls

- Two private buckets: `respondent-inputs`, `respondent-renders`.
- Opaque object paths; no user-provided filenames.
- Service/server creates signed upload/download URLs with short TTL.
- Owner never receives participant object paths or signed URLs.
- Participant gets a signed result URL only after cookie token/session ownership check.
- Supabase service-role key is server-only; no anon direct write to participant data tables.
- RLS enabled for every DB table.

## 6. Image security

- Accept JPEG/JPG only in MVP.
- Max upload 10MB before decode; server-side signature and decoder validation.
- Re-encode with Sharp to normalised JPEG, max 1024px long side and strip EXIF.
- Reject animated formats, SVG, PDFs, archives and arbitrary URLs.
- Limit decoded pixel count to protect memory/CPU; set processing timeout.
- Run image processing in server worker, not browser or request handler.
- Do not use images to train Groom or any model.
- Verify Perfect Corp's permitted use and retention in current terms before upload; do not infer provider privacy guarantees from API docs.

## 7. Access control

- Owner can see own experiments and aggregate results only.
- Owner cannot inspect participant rows, image paths, renders, or raw votes.
- Public participant page sees public experiment metadata only.
- Participant token is random, HMAC-hashed at rest, cookie-protected and short-lived.
- No public database table access for participant state.
- Operator access uses MFA and is logged where possible.

## 8. Incident controls

- `GROOM_NEW_SESSIONS_ENABLED=false`: stops new sessions while existing sessions/cleanup continue.
- `GROOM_RENDER_ENABLED=false`: stops new VTO tasks.
- Rotate YouCam, Supabase, Inngest and HMAC secrets if exposed; invalidate participant sessions if token secret compromised.
- Pause affected experiment; preserve redacted logs; assess which images/records were affected; follow legal notification obligations after counsel/privacy lead review.
- Never send participant photos or signed URLs in support tickets unless absolutely necessary and securely handled.

## 9. Launch blockers

- Confirm Perfect Corp data processing terms, retention, data location, sub-processors, deletion and secondary use.
- Publish accurate Groom Privacy Notice and Terms.
- Establish privacy contact and request/complaint handling procedure.
- Review cross-border data transfer and data-subject rights obligations for target markets.
- Confirm vendor terms allow the chosen research/commercial use of VTO outputs and template thumbnails.
