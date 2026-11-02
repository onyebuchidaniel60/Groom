# Groom — Planned Repository Structure

The following structure is the implementation target. It is a planned structure, not a claim that these files already exist.

```text
groom/
├── PROJECT_SPEC.md
├── ARCHITECTURE.md
├── IMPLEMENTATION_PLAN.md
├── AGENTS.md
├── AI_HANDOFF.md
├── README.md
├── .env.example
├── .gitignore
├── package.json
├── package-lock.json
├── next.config.ts
├── tsconfig.json
├── eslint.config.mjs
├── postcss.config.mjs
├── playwright.config.ts
├── vitest.config.ts
├── src/
│   ├── app/
│   │   ├── (public)/
│   │   │   ├── page.tsx
│   │   │   ├── docs/page.tsx
│   │   │   ├── privacy/page.tsx
│   │   │   ├── terms/page.tsx
│   │   │   └── support/page.tsx
│   │   ├── (auth)/
│   │   │   ├── sign-up/page.tsx
│   │   │   └── sign-in/page.tsx
│   │   ├── (owner)/
│   │   │   └── dashboard/
│   │   │       ├── page.tsx
│   │   │       └── experiments/
│   │   │           ├── new/page.tsx
│   │   │           └── [id]/
│   │   │               ├── page.tsx
│   │   │               └── results/page.tsx
│   │   ├── p/[slug]/page.tsx
│   │   ├── api/
│   │   │   ├── health/route.ts
│   │   │   ├── style-templates/route.ts
│   │   │   ├── experiments/route.ts
│   │   │   ├── experiments/[id]/route.ts
│   │   │   ├── experiments/[id]/publish/route.ts
│   │   │   ├── experiments/[id]/pause/route.ts
│   │   │   ├── experiments/[id]/resume/route.ts
│   │   │   ├── experiments/[id]/close/route.ts
│   │   │   ├── experiments/[id]/archive/route.ts
│   │   │   ├── experiments/[id]/results/route.ts
│   │   │   ├── public/experiments/[slug]/route.ts
│   │   │   ├── public/experiments/[slug]/sessions/route.ts
│   │   │   ├── participant/upload-url/route.ts
│   │   │   ├── participant/upload-complete/route.ts
│   │   │   ├── participant/session/route.ts
│   │   │   ├── participant/submit/route.ts
│   │   │   └── inngest/route.ts
│   │   ├── layout.tsx
│   │   ├── not-found.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── layout/
│   │   ├── auth/
│   │   ├── experiments/
│   │   ├── participant/
│   │   ├── results/
│   │   └── ui/
│   ├── lib/
│   │   ├── config/
│   │   ├── supabase/
│   │   ├── validation/
│   │   ├── errors/
│   │   └── utils/
│   ├── server/
│   │   ├── auth/
│   │   ├── domain/
│   │   │   ├── experiments/
│   │   │   ├── participants/
│   │   │   ├── responses/
│   │   │   └── usage/
│   │   ├── repositories/
│   │   ├── integrations/youcam/
│   │   ├── jobs/
│   │   ├── storage/
│   │   ├── security/
│   │   └── services/
│   └── types/
│       ├── database.ts
│       └── api.ts
├── supabase/
│   ├── config.toml
│   ├── migrations/
│   └── seed.sql
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── api/
│   ├── e2e/
│   ├── fixtures/
│   └── security/
├── docs/
│   ├── OPERATOR_GUIDE.md
│   ├── USER_GUIDE.md
│   ├── YOUCAM_INTEGRATION.md
│   ├── SECURITY_PRIVACY.md
│   ├── TEST_STRATEGY.md
│   ├── PRODUCT_RESEARCH.md
│   ├── DESIGN_APPROVAL.md
│   ├── DECISION_LOG.md
│   ├── EDGE_CASE_MATRIX.md
│   ├── REPOSITORY_STRUCTURE.md
│   ├── PHASE0_REPORT.md
│   ├── RELEASE_CHECKLIST.md
│   ├── INCIDENT_RESPONSE.md
│   └── design/
└── .github/
    └── workflows/ci.yml
```

## Directory ownership

- `src/app`: route/page definitions and thin route handlers.
- `src/components`: UI composition only; no privileged provider or DB secrets.
- `src/lib`: shared safe utilities and client/server adapters.
- `src/server/domain`: pure business rules and state transitions.
- `src/server/repositories`: DB access, with explicit owner/session-scoped methods.
- `src/server/integrations/youcam`: provider schemas, adapter and response normalisation.
- `src/server/jobs`: Inngest functions and scheduled cleanup.
- `supabase/migrations`: authoritative, versioned schema and RLS.
- `tests`: tests grouped by concern; no production personal data in fixtures.
- `docs/design`: approved references/mockups only after operator approval.

Keep route handlers thin: authenticate, validate, call domain/service, map response. Keep SQL/authorization logic out of client components.
