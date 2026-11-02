# Groom — Exact First Coding-Agent Task

**Assigned phase:** Phase 0, task 1 only.  
**Task type:** Read-only reconnaissance.  
**Do not implement or modify anything in this task.**

```text
Read AGENTS.md, PROJECT_SPEC.md, ARCHITECTURE.md, IMPLEMENTATION_PLAN.md and AI_HANDOFF.md first.

Your only task is to inspect the existing Groom repository and connected development/deployment setup without modifying files, installing dependencies, running migrations, creating API tasks, changing settings, or deploying anything.

Inspect and report:
1. Current repository URL/remotes (do not invent one), branch, commit SHA, and clean/dirty working-tree status.
2. Existing directory structure, README/docs, package manager/lockfile, package.json, Node version/configuration and app entry points.
3. Existing Next.js/React/TypeScript/Tailwind versions and whether the repo already contains work that must be preserved.
4. Existing database/Supabase integration, migrations, RLS policies, storage buckets and auth wiring. Inspect configuration and names only; do not print credential values.
5. Existing Vercel project/Preview settings, domains, deployment protection and environment variable NAMES only. Do not reveal secret values.
6. Existing Inngest setup and event route, if any.
7. Which required environment variable names are present or missing, without exposing values.
8. Whether a public GitHub repository URL exists and whether it is actually public. Do not add a footer link.
9. Any direct conflict between the existing repository and the approved Groom blueprint.
10. What Phase 0 checks remain for the YouCam account: active API key, redeemed unit balance, current hairstyle/beard template IDs, live test approval, task polling retention, failure billing and provider data-retention/privacy terms. Do not call the provider or create a billable task without explicit operator authorisation.

Do not create a new Next.js app, scaffold routes, edit documentation, change package versions, add environment variables, alter database schemas, seed templates, or deploy in this task.

Return:
- Summary of what exists.
- Exact repository and deployment facts.
- Relevant files inspected.
- Missing/unknown access and environment configuration.
- Risks and conflicts with the blueprint.
- Whether the repository is ready for Phase 1 or needs operator action first.

Stop after the report and wait. Do not begin Phase 0 task 2 or Phase 1 automatically.
```

The agent must update `AI_HANDOFF.md` only after the operator accepts the reconnaissance report; this first task is otherwise read-only.
