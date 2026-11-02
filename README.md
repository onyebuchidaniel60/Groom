# Groom

Groom is a virtual consumer-testing platform for men's grooming brands and barbershop businesses. It helps a business test a proposed **hairstyle collection or beard-style collection** with consenting adult participants before deciding what to launch or promote.

> This repository is a blueprint handoff package. The application has not been implemented. Do not treat this repository/document bundle as a working product.

## Read first

Start with [`GROOM_IMPLEMENTATION_BLUEPRINT.md`](./GROOM_IMPLEMENTATION_BLUEPRINT.md) for the complete package index.

1. [`PROJECT_SPEC.md`](./PROJECT_SPEC.md) — product definition, MVP, flows, features and business rules.
2. [`ARCHITECTURE.md`](./ARCHITECTURE.md) — fixed architecture, database, APIs, security and integrations.
3. [`IMPLEMENTATION_PLAN.md`](./IMPLEMENTATION_PLAN.md) — phase-by-phase implementation and stop gates.
4. [`AGENTS.md`](./AGENTS.md) — persistent coding-agent instructions.
5. [`AI_HANDOFF.md`](./AI_HANDOFF.md) — current phase, outstanding checks and next task.

## Current status

- Product: selected.
- Novelty validation: not yet passed; do not claim this is the first or only product of its kind.
- Repository/application code: not started.
- YouCam live account and redeemed balance: must be confirmed by operator.
- Preview URL: none yet.
- Final visual design: not chosen or approved.

## Planned stack

Next.js App Router, React, TypeScript, Tailwind, Zod, Supabase PostgreSQL/Auth/private Storage, Inngest, Perfect Corp YouCam REST APIs, `sharp`, Vitest, React Testing Library, Playwright and Vercel. See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for exact responsibilities and constraints.

## Planned local setup (after implementation)

The coding agent must fill in exact, verified steps and package scripts during Phase 1. Do not invent commands or package versions before the app exists.

Expected high-level setup:

1. Install the documented Node.js 22 LTS version.
2. Clone the actual project repository.
3. Install dependencies using the committed lockfile and package manager.
4. Copy `.env.example` to `.env.local` and supply safe development credentials.
5. Start local Supabase or connect to the isolated development Supabase project.
6. Apply migrations and seed only curated, verified template metadata.
7. Start Next.js and Inngest development services using the documented commands.
8. Run tests, lint, typecheck and production build.

## Environment variables

See [`ARCHITECTURE.md §14`](./ARCHITECTURE.md#14-environment-variables) and `.env.example` once created. Never commit secrets. Do not put any secret in a `NEXT_PUBLIC_` variable.

## Database and storage

Migrations belong in `supabase/migrations/`. RLS is mandatory on every application table. Image buckets are private. No participant images are ever exposed through a public object URL.

## Testing

See [`docs/TEST_STRATEGY.md`](./docs/TEST_STRATEGY.md). A feature is not complete because code was generated; it is complete only after its acceptance criteria and verification steps pass.

## Deployment

The implementation must deploy a skeletal, navigable frontend to a stable Vercel Preview during Phase 1. Preview URL and deployed commit are recorded in `AI_HANDOFF.md` at every accepted checkpoint. Production deployment requires explicit operator approval.

## Documentation

- [`docs/OPERATOR_GUIDE.md`](./docs/OPERATOR_GUIDE.md)
- [`docs/USER_GUIDE.md`](./docs/USER_GUIDE.md)
- [`docs/YOUCAM_INTEGRATION.md`](./docs/YOUCAM_INTEGRATION.md)
- [`docs/SECURITY_PRIVACY.md`](./docs/SECURITY_PRIVACY.md)
- [`docs/TEST_STRATEGY.md`](./docs/TEST_STRATEGY.md)
- [`docs/API_SPECIFICATION.md`](./docs/API_SPECIFICATION.md)
- [`docs/EDGE_CASE_MATRIX.md`](./docs/EDGE_CASE_MATRIX.md)
- [`docs/REPOSITORY_STRUCTURE.md`](./docs/REPOSITORY_STRUCTURE.md)
- [`docs/PRE_IMPLEMENTATION_AUDIT.md`](./docs/PRE_IMPLEMENTATION_AUDIT.md)
- [`docs/REFERENCE_SOURCES.md`](./docs/REFERENCE_SOURCES.md)
- [`docs/ENVIRONMENT_CONFIGURATION.md`](./docs/ENVIRONMENT_CONFIGURATION.md)
- [`docs/CODING_AGENT_START_TASK.md`](./docs/CODING_AGENT_START_TASK.md)
- [`docs/PROPER_VIBE_CODING_WORKFLOW.md`](./docs/PROPER_VIBE_CODING_WORKFLOW.md) — original workflow reference supplied by operator.
- [`docs/PRODUCT_RESEARCH.md`](./docs/PRODUCT_RESEARCH.md)
- [`docs/DESIGN_APPROVAL.md`](./docs/DESIGN_APPROVAL.md)
- [`docs/DECISION_LOG.md`](./docs/DECISION_LOG.md)

## Important limits

Groom's results are directional feedback from the people who participated. They are not a representative market survey, scientific prediction, clinical finding or guarantee of physical haircut/beard outcome. AI-generated previews are visual simulations and may differ from real barber execution.
