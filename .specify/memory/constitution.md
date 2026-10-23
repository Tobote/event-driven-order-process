# Event-Driven Order Process Constitution

## Core Principles

### I. Spec before code
Every feature starts as a spec in `specs/NNN-feature/`: spec.md, then plan.md, then tasks.md. Specs describe WHAT and WHY. Plans describe HOW. Code that has no task to trace back to is not merged.

### II. Test-first (NON-NEGOTIABLE)
Tests are written from the acceptance criteria before the implementation, and they must fail first (red → green → refactor). Each user story has at least one automated acceptance/integration test. Unit tests cover domain logic. Tests run in CI and locally with one command.

### III. Simplicity and YAGNI
Choose the simplest design that meets the current spec. No speculative abstractions, layers or configuration. Any added complexity (a new service, a new technology, a pattern beyond the obvious) must be justified in the plan's Complexity Tracking table or an ADR.

### IV. Explicit contracts
Every boundary between components (HTTP APIs, messages and events, public module interfaces) has a written contract under `specs/NNN-feature/contracts/`. Breaking changes need a version bump and a migration note. Contract tests protect them.

### V. Security by default
No secrets in code, logs or git. Configuration comes from the environment, and `.env.example` documents it. Validate all external input at the boundary. Use least privilege for credentials. Dependencies are pinned and kept current.

### VI. Observability
Use structured logs (JSON) with a correlation ID that propagates across every call and message. Expose health/readiness endpoints for each runnable service. Errors are logged once, with context, at the boundary that handles them.

### VII. [PROJECT_PRINCIPLE_7_NAME]
<!-- Project-specific, to be filled by /speckit-constitution. Suggested: "Reliable messaging" - transactional outbox for every publish, idempotent consumers, retries with backoff + DLQ, saga with compensation for multi-service workflows. -->
[PROJECT_PRINCIPLE_7_DESCRIPTION]

### VIII. [PROJECT_PRINCIPLE_8_NAME]
<!-- Project-specific, to be filled by /speckit-constitution. Suggested: "Service autonomy" - one database per service, integration only via messages/APIs, `docker compose up` runs the whole system. -->
[PROJECT_PRINCIPLE_8_DESCRIPTION]

## Technology & Architecture Constraints
<!-- To be ratified by /speckit-constitution. Proposed: C#/.NET LTS for Order/Payment/Inventory (Minimal APIs, EF Core, MassTransit); Node.js/TypeScript for Notification and API gateway; RabbitMQ as primary broker (Kafka only via ADR); PostgreSQL per service; Redis for idempotency/cache; Docker Compose for local env. -->
[TECH_CONSTRAINTS_CONTENT]

## Development Workflow & Quality Gates
- Flow per feature: `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` (+ `architect` agent review) → `/speckit-tasks` → `/speckit-analyze` → `/speckit-implement` → `code-reviewer` agent → PR.
- A PR is mergeable only when all tests pass, `/speckit-analyze` reports no critical issues, and the review findings are addressed.
- One branch per spec, Conventional Commits that reference task IDs.

## Governance
- This constitution overrides individual preferences. Amendments are recorded with a semver bump: MAJOR = a principle removed or redefined, MINOR = a principle added, PATCH = wording.
- `/speckit-analyze` and the `architect` agent check plans against this constitution. Violations must be justified in the plan's Complexity Tracking.
- Significant architectural decisions are recorded as ADRs in `docs/adr/`.
- Runtime development guidance lives in `CLAUDE.md`.

**Version**: 0.1.0 | **Ratified**: 2026-10-22 | **Last Amended**: 2026-10-22
