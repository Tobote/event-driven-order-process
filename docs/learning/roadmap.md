# Study roadmap: Event-Driven Order Processing Platform

**Goal:** learn event-driven architecture, Saga, Outbox, eventual consistency and distributed transactions by building this system **yourself**. Claude tutors and reviews; you write the code.

**Pace:** ~3–5 h/week in 1–2 h sessions, about 15 weeks.

## Method
For every module: **Learn → Decide → Specify → Build → Break → Review → Check.**

> **Build it naively, break it on purpose, then fix it with the pattern.**
> You feel each problem before you learn its solution.

### Session template (1–2 h)
1. Ask Claude to start the session (5 min). It reads the journal and proposes today's goal.
2. Concept or reading for the current module (~20 min).
3. Build one or two tasks from `tasks.md`, test-first (~60 min).
4. `mentor-reviewer` review of what you wrote (~15 min).
5. Journal entry with the `learning-journal` skill (5 min).

### SDD flow per feature (study mode)
1. **Learn:** Claude explains the module's concepts.
2. **`/speckit-specify`:** you describe the feature in your own words. Claude asks what you missed.
3. **`/speckit-clarify`:** you answer, as the product owner.
4. **`decision-coach` → ADR:** you choose and write `docs/adr/NNNN-*.md`.
5. **`/speckit-plan` + `/speckit-tasks`:** review them, and explain each choice back to Claude. Ask the `sdd-core:architect` agent for a second opinion.
6. **Build:** you implement the tasks, test-first. Claude gives hints only.
7. **Break it:** run the module's failure experiment and note what happened.
8. **`mentor-reviewer` → `concept-check` → journal → PR/merge.**

---

## Modules

### ☐ Module 0: Foundations & environment (weeks 1–2)
- **Learn:** sync vs async communication, commands vs events, temporal and spatial coupling, CAP intuition, what "eventual consistency" really means.
- **Read:** *Microservices Patterns* ch. 3 (messaging); RabbitMQ tutorials 1–3.
- **Decide:** ratify the constitution (`/speckit-constitution`), then ADR **0001 broker: RabbitMQ vs Kafka**.
- **Build:** install the .NET SDK and Docker Desktop. Write `docker-compose.yml` with Postgres, RabbitMQ and Redis, plus health checks.
- **Break:** stop a container and watch `docker compose ps` and the health checks react.
- **Done when:** `docker compose up -d` is all healthy, constitution v1.0 is ratified, and ADR 0001 is accepted.
- [ ] Constitution ratified · [ ] ADR 0001 · [ ] compose healthy · [ ] concept check ≥ 80% · [ ] journal

### ☐ Module 1: Place order, the naive way (weeks 3–4)
- **Learn:** service boundaries, one database per service, the **dual-write problem**.
- **Decide:** ADR **0002 service split & data ownership** (which services own which data).
- **Spec:** `001-place-order`. Order service (.NET): `POST /orders` → save to Postgres → publish `OrderPlaced` **directly** to RabbitMQ.
- **Break:** kill the process (or the broker) between save and publish. The order exists, but no event was sent.
- **Done when:** tests are green and you can explain exactly why this design is unsafe.
- [ ] Spec · [ ] ADR 0002 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 2: Transactional Outbox (weeks 5–6)
- **Learn:** the outbox pattern, at-least-once delivery, polling relay vs CDC (Debezium), why duplicates are now unavoidable.
- **Read:** microservices.io *Transactional Outbox*; *Microservices Patterns* ch. 3.3.
- **Decide:** ADR **0003 own relay vs MassTransit outbox**. Recommendation for learning: build your own first, then compare.
- **Spec:** `002-reliable-publishing`. Outbox table, written in the same transaction as the order, plus a background relay.
- **Break:** stop RabbitMQ while placing orders, start it again, and confirm nothing was lost. Then kill the relay after publish and before marking the row sent. What happens?
- **Done when:** the experiment is repeatable and nothing is lost, and you can explain where the duplicates come from.
- [ ] Spec · [ ] ADR 0003 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 3: Consumers, idempotency, retries & DLQ (weeks 7–8)
- **Learn:** the idempotent consumer / inbox, retries with backoff and jitter, dead-letter queues, poison messages, ordering guarantees (or the lack of them).
- **Read:** microservices.io *Idempotent Consumer*; *Enterprise Integration Patterns*, Dead Letter Channel.
- **Decide:** ADR **0004 retry & DLQ policy**.
- **Spec:** `003-inventory-reservation`. Inventory service (.NET) consumes `OrderPlaced`, reserves stock, and publishes `StockReserved` or `StockRejected`.
- **Break:** deliver the same message twice, throw an exception inside the handler, publish a malformed message.
- **Done when:** there are no double reservations, transient failures recover, and poison messages land in the DLQ.
- [ ] Spec · [ ] ADR 0004 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 4: Polyglot consumer & contracts (week 9)
- **Learn:** message contracts, schema evolution (additive vs breaking changes), cross-language interop, Redis idempotency keys.
- **Decide:** ADR **0005 message envelope format** (MassTransit envelope vs raw JSON).
- **Spec:** `004-order-notifications`. Notification service (Node/TS) consumes order events and "sends" notifications (logs or a fake email).
- **Break:** add a new field to an event (should be fine), then rename a field (it breaks). See how the contract test catches it.
- **Done when:** a contract test passes in both the .NET and Node codebases.
- [ ] Spec · [ ] ADR 0005 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 5: Saga & compensation, distributed transactions (weeks 10–12)
- **Learn:** why not 2PC (blocking, coordinator failure, availability), the Saga pattern, orchestration vs choreography, compensating actions, timeouts, semantic locks (`PENDING` states), the countermeasures for lack of isolation.
- **Read:** *Microservices Patterns* ch. 4; *DDIA* ch. 9 (atomic commit and 2PC); the MassTransit saga state machine docs.
- **Decide:** ADR **0006 orchestration vs choreography**.
- **Spec:** `005-payment` (Payment service, .NET, with a configurable failure rate) and `006-order-saga` (the state machine: reserve stock → charge payment → confirm order, with compensations).
- **Draw first:** sequence diagrams for the happy path, payment failure, payment timeout and stock rejection, *before* coding.
- **Break:** payment fails, payment times out, a compensation step itself fails, a service restarts mid-saga.
- **Done when:** every path ends in a consistent state that matches your diagrams.
- [ ] Diagrams · [ ] Spec 005 · [ ] Spec 006 · [ ] ADR 0006 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 6: Eventual consistency in practice (week 13)
- **Learn:** read models (CQRS-lite), `202 Accepted` + status polling, cache invalidation by events, correlation IDs, distributed tracing.
- **Decide:** ADR **0007 read-model approach**.
- **Spec:** `007-order-status`. Order status API, Redis cache, OpenTelemetry tracing (Compose `observability` profile).
- **Break:** read the status immediately after placing an order and get a stale read. Explain it and design the UX for it.
- **Done when:** one trace in Grafana/Tempo shows the full saga across services.
- [ ] Spec · [ ] ADR 0007 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 7 (optional): Kafka & event streaming (weeks 14–15)
- **Learn:** log vs queue, partitions and keys, consumer groups, offsets, replay, compaction.
- **Read:** *DDIA* ch. 11.
- **Decide:** ADR **0008 why Kafka here**.
- **Spec:** `008-order-events-stream`. Publish order events to Kafka, plus an analytics consumer (Node or .NET).
- **Break:** reset the consumer offsets and replay. What happens to the consumer's side effects?
- **Done when:** you can explain when you'd choose RabbitMQ vs Kafka, with examples from this project.
- [ ] Spec · [ ] ADR 0008 · [ ] Build · [ ] Break · [ ] Review · [ ] Concept check · [ ] journal

### ☐ Module 8: Wrap-up (week 15)
- A chaos session: randomly kill services and brokers during load, and verify consistency.
- Write the architecture README: diagrams, decisions and lessons learned.
- Retrospective: turn what you learned into updates to the `claude-ai-base` skills.
- Final `concept-check` across all modules, ≥ 80%.
- [ ] Chaos · [ ] README · [ ] Retro → ai-base · [ ] Final check

---

## Core references
- Chris Richardson, *Microservices Patterns* (Manning): ch. 3 messaging, ch. 4 sagas. Also https://microservices.io/patterns/
- Martin Kleppmann, *Designing Data-Intensive Applications*: ch. 9 and ch. 11
- Hohpe & Woolf, *Enterprise Integration Patterns*: https://www.enterpriseintegrationpatterns.com/
- RabbitMQ tutorials: https://www.rabbitmq.com/tutorials
- MassTransit docs: https://masstransit.io/documentation (consumers, outbox, sagas)
