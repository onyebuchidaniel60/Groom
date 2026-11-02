# Groom — Complete Implementation Blueprint

This is the master navigation document for the Groom blueprint package. The substantive implementation-ready decisions are contained in the five core handoff files and the supporting `docs/` material. No application code is included.

## Required reading order

1. [Product definition, MVP, roles, user flows, requirements and business logic](./PROJECT_SPEC.md)
2. [Data model, system architecture, stack, API, transactions, auth, security, integrations and UI architecture](./ARCHITECTURE.md)
3. [Phased implementation sequence and acceptance gates](./IMPLEMENTATION_PLAN.md)
4. [Persistent coding-agent rules](./AGENTS.md)
5. [Current handoff state and next task](./AI_HANDOFF.md)

## Supporting documentation

- [API contract](./docs/API_SPECIFICATION.md)
- [YouCam provider integration](./docs/YOUCAM_INTEGRATION.md)
- [Security and privacy](./docs/SECURITY_PRIVACY.md)
- [Test strategy](./docs/TEST_STRATEGY.md)
- [Error and edge-case matrix](./docs/EDGE_CASE_MATRIX.md)
- [Repository structure](./docs/REPOSITORY_STRUCTURE.md)
- [Environment configuration](./docs/ENVIRONMENT_CONFIGURATION.md)
- [Operator guide](./docs/OPERATOR_GUIDE.md)
- [Participant/business user guide](./docs/USER_GUIDE.md)
- [Product and novelty research](./docs/PRODUCT_RESEARCH.md)
- [Design reference and approval gate](./docs/DESIGN_APPROVAL.md)
- [Decision log](./docs/DECISION_LOG.md)
- [Phase 0 report template](./docs/PHASE0_REPORT.md)
- [Pre-implementation audit](./docs/PRE_IMPLEMENTATION_AUDIT.md)
- [Release checklist](./docs/RELEASE_CHECKLIST.md)
- [Incident response](./docs/INCIDENT_RESPONSE.md)
- [Reference sources](./docs/REFERENCE_SOURCES.md)
- [Original Proper Vibe Coding Workflow supplied by operator](./docs/PROPER_VIBE_CODING_WORKFLOW.md)
- [Exact coding-agent start task](./docs/CODING_AGENT_START_TASK.md)

## Key gates

- Groom's novelty status is **NOT READY**; the product direction is selected but no provisional novelty pass is claimed.
- Phase 0 provider/account/privacy checks must pass before live photo processing.
- Phase 1 must deploy the early skeletal frontend and report its exact Preview URL/commit.
- Critical MVP journeys must pass end-to-end tests before final visual design.
- Design references and mockups must be presented; final UI requires explicit operator approval.
- Final frontend/docs/footer/link completeness is a deliverable.
- Production release requires explicit operator approval.

See [MANIFEST.md](./MANIFEST.md) for package status and [AI_HANDOFF.md](./AI_HANDOFF.md) for the precise first task.
