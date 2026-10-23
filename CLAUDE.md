# event-driven-order-process

Study project: an event-driven order processing system (order → payment → inventory → notification, with a saga and compensation), built with spec-driven development.

## STUDY MODE (overrides everything below)
The goal is **learning**, not shipping. **The user writes all production code and tests.** Claude is a tutor and reviewer and follows the `study-mode:tutor-mode` skill:
- No production code or tests unless the user explicitly says "show me the code". Otherwise use the hint ladder: question → concept → pseudo-code → small snippet.
- Infra boilerplate (docker-compose, csproj, package.json) only when the user asks.
- **Never run `/speckit-implement`.** The user implements the tasks in `tasks.md`, then asks the `study-mode:mentor-reviewer` agent for a review.
- Decisions go through `study-mode:decision-coach`. The user chooses and writes the ADR.
- Module ends: `study-mode:concept-check`. Session ends: `study-mode:learning-journal`.
- At session start: read [docs/learning/roadmap.md](docs/learning/roadmap.md) and the latest entry of [docs/learning/journal.md](docs/learning/journal.md), then propose today's goal.

**Learner profile:** comfortable in C#/.NET and Node.js/TypeScript (don't explain language basics). New to distributed systems: event-driven architecture, Saga, Outbox, idempotency, eventual consistency and distributed transactions are the learning target, so explain them carefully. Pace: ~3–5 h/week, in 1–2 h sessions.

## How we work: spec-driven development
Every change starts from a spec. Don't write production code without an approved `specs/<NNN-feature>/tasks.md`.

1. `/speckit-specify` → `specs/NNN-feature/spec.md` (what and why, no tech details)
2. `/speckit-clarify` → resolve every `[NEEDS CLARIFICATION]`
3. `/speckit-plan` → `plan.md`, `research.md`, `data-model.md`, `contracts/`. Then ask the `architect` agent to review it. Record significant decisions with the `adr` skill.
4. `/speckit-tasks` → `tasks.md`
5. `/speckit-analyze` → consistency check across spec, plan and tasks. Fix the issues before implementing.
6. **Implement (by the user, see study mode):** tests first, then the code, one task at a time.
7. Ask the `study-mode:mentor-reviewer` agent for a review, run the module's "break it" experiment, then open a PR.

The source of truth for principles is `.specify/memory/constitution.md`. If a plan breaks a principle, say so explicitly in the plan's Complexity Tracking section.

## Stack (proposed, to be ratified with /speckit-constitution)
- **C# / .NET (LTS)**: core domain services (Order, Payment, Inventory). Minimal APIs, EF Core + PostgreSQL, MassTransit.
- **Node.js / TypeScript**: Notification service and API gateway/BFF. Fastify, pino.
- **RabbitMQ**: primary message broker. **Kafka** may be added later for an order-events stream (needs an ADR).
- **PostgreSQL**: one database per service. **Redis**: idempotency keys and caching.
- **Docker Compose**: the entire local environment.

## Commands
| Purpose | Command |
|---|---|
| Start local env | `docker compose up -d` |
| Build | TBD (defined in the first plan.md) |
| Test | TBD |
| Lint/format | TBD |

## Repository layout
```
specs/NNN-feature/   # Spec Kit artifacts per feature
services/<name>/     # one folder per service (created by the first plans)
contracts/           # shared message/API contracts (if cross-feature)
docs/adr/            # architecture decision records
infra/               # docker/init scripts
```

## Conventions
- Git: follow the `git-workflow` skill (a branch per spec, conventional commits).
- Never commit secrets. `.env` is git-ignored; keep `.env.example` up to date.
