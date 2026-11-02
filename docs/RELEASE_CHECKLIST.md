# Groom — Release Checklist

## Product and API
- [ ] Current YouCam API schemas and template IDs verified.
- [ ] One real hairstyle and one real beard task tested.
- [ ] Unit budget reconciled against provider console.
- [ ] No ambiguous provider task creation remains unreconciled.
- [ ] Participant flow requires explicit consent and 18+ affirmation.
- [ ] Owner receives aggregate results only.

## Data and security
- [ ] RLS enabled and cross-owner tests pass.
- [ ] Private storage buckets and signed URL expiry verified.
- [ ] Image re-encoding and EXIF stripping verified.
- [ ] No participant images, cookies, signed URLs or secrets in logs.
- [ ] Retention/cleanup job verified and orphan objects reconciled.
- [ ] Provider terms/retention/data transfer reviewed.
- [ ] Privacy notice/terms reviewed for operating jurisdictions.
- [ ] Kill switches tested.

## Functional quality
- [ ] Owner creates and publishes hairstyle experiment.
- [ ] Participant completes hairstyle flow.
- [ ] Owner sees correct aggregated results.
- [ ] Owner creates and publishes beard experiment.
- [ ] Participant completes beard flow.
- [ ] Duplicate submit, concurrency, close and expiry tests pass.
- [ ] All critical E2E tests pass on Preview.

## Frontend completeness
- [ ] All routes, nav and footer are correct.
- [ ] Mobile/tablet/desktop responsive checks pass.
- [ ] Keyboard, focus, labels and contrast basics pass.
- [ ] Loading/empty/error/success/expired/permission/not-found states complete.
- [ ] Documentation link points to real `/docs`.
- [ ] GitHub link points to actual approved public repository, or private-repo exception is documented and approved.
- [ ] No dead links, fake URLs, inert controls or unexplained placeholders.

## Final UI governance
- [ ] Core MVP E2E gate passed before final visual design.
- [ ] Design references collected/approved.
- [ ] Mockups presented and revised.
- [ ] Explicit operator approval recorded in `docs/DESIGN_APPROVAL.md`.
- [ ] Final UI matches approved mockups without business/architecture changes.

## Release
- [ ] Lint, typecheck, unit, integration, E2E and production build pass.
- [ ] Dependency/security audit complete.
- [ ] Migration dry run and backup/restore plan checked.
- [ ] Production variables isolated from Preview.
- [ ] Operator explicitly approved production deployment.
- [ ] Production smoke test and monitoring active.
