<!--
Sync Impact Report
- Version change: 0.1.0 → 1.0.0 (first ratification; draft placeholders replaced, governance redefined for study mode)
- Modified principles:
  - II. Test-first: unchanged in intent; confirmed by the user ("test first")
  - VII. [PROJECT_PRINCIPLE_7_NAME] → VII. Reliable messaging
  - VIII. [PROJECT_PRINCIPLE_8_NAME] → VIII. Data ownership
- Added sections:
  - Purpose (preamble)
  - IX. Asynchronous service-to-service communication
  - X. Understand before abstracting
  - Technology & Architecture Constraints (filled)
- Removed sections: none
- Changed: Development Workflow (user implements, no /speckit-implement; mentor-reviewer instead of code-reviewer)
- Follow-up TODOs:
  - Principles I, III, IV, V, VI come from the bootstrap draft and were not discussed in the ratification
    session; the user should confirm or amend them.
-->
# Event-Driven Order Process Constitution

## Purpose

This is a study project. Its goal is to learn event-driven architecture by building an order
processing system (order → payment → inventory → notification, with a saga and compensation):
temporal and spatial coupling, commands vs events, producers and consumers, eventual consistency,
compensation and sagas. When learning and shipping speed conflict, learning wins.

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

### VII. Reliable messaging
- **Producers** MUST guarantee that a state change is saved AND its message is published, never
  one without the other. This applies from Module 2 onward. Module 1 deliberately publishes
  directly after saving to demonstrate the dual-write problem, and its plan records that.
- **Consumers** MUST be idempotent: processing the same message twice MUST NOT repeat its side
  effects.
- **Consumers** MUST handle failures: transient errors are retried, and permanent failures go to a
  dead-letter queue or result in a published failure event. A message is never silently lost.

Rationale: brokers deliver at least once, and processes crash between any two steps.

### VIII. Data ownership
Each service owns its database. No service reads or writes another service's database. A service
that needs another service's data gets it from published events and keeps its own copy (a read
model) where needed.

Rationale: a shared database couples services at the schema level and hides who owns the truth.

### IX. Asynchronous service-to-service communication
Service-to-service communication in business flows MUST be asynchronous, through messages on the
broker. Synchronous HTTP is allowed only for clients at the edge (client or gateway/BFF → service).

Rationale: synchronous chains create temporal coupling, so every service must be up at the same
moment for the flow to succeed.

### X. Understand before abstracting
Before using a framework feature that implements a pattern (for example MassTransit's outbox or
saga state machines), the developer MUST first understand how the pattern works, by building or
explaining a minimal version of it. The ADR that adopts the framework feature MUST say what it
does under the hood.

Rationale: the goal of the project is to learn the patterns, not only to use them.

## Technology & Architecture Constraints

- **C# / .NET (LTS)** for the Order, Payment and Inventory services: Minimal APIs, EF Core with
  PostgreSQL, MassTransit.
- **Node.js / TypeScript** for the Notification service and the API gateway/BFF: Fastify, pino.
- **RabbitMQ** is the primary message broker. Kafka may be added only through an ADR.
- **PostgreSQL**, one database per service (see VIII). **Redis** for idempotency keys and caching.
- **Docker Compose** runs the entire local environment.

## Development Workflow & Quality Gates

- Flow per feature: `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` (+ `architect` agent review) → `/speckit-tasks` → `/speckit-analyze` → implementation by the user, test-first → `mentor-reviewer` agent → "break it" experiment → PR.
- Study mode: the user writes all production code and tests. `/speckit-implement` is not used.
- A PR is mergeable only when all tests pass, `/speckit-analyze` reports no critical issues, and the review findings are addressed.
- One branch per spec, Conventional Commits that reference task IDs.

## Governance

- This constitution overrides individual preferences. Amendments are recorded with a semver bump: MAJOR = a principle removed or redefined, MINOR = a principle added, PATCH = wording.
- `/speckit-analyze` and the `architect` agent check plans against this constitution. Violations must be justified in the plan's Complexity Tracking.
- Significant architectural decisions are recorded as ADRs in `docs/adr/`, written by the user.
- Runtime development guidance lives in `CLAUDE.md`.

**Version**: 1.0.0 | **Ratified**: 2026-10-23 | **Last Amended**: 2026-10-23
