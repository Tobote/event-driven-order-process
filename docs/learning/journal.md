# Learning journal

One entry per study session, newest first. Ask Claude to "log this session" (the `learning-journal` skill) at the end of each session.

Entry format:
```markdown
### YYYY-MM-DD · Module N · <short title>
- **Did:** ...
- **Learned:** (in your own words)
- **Surprised / broke:** ...
- **Open questions:** ...
- **Next:** ...
- **Time:** ~Xh
```

## Entries

### 2026-10-26 · Module 0 · Concepts + ADR 0001 (message broker)
- **Did:** Went through the Module 0 concepts with exercises: sync vs async, temporal and spatial coupling, commands vs events (5/5), CAP, and eventual consistency. Ran the decision-coach for the broker choice: framed the question, listed forces, compared RabbitMQ, Kafka, Redis, Azure Service Bus and Postgres-as-queue. Wrote and committed [ADR 0001: Use RabbitMQ as the message broker](../adr/0001-message-broker.md).
- **Learned:** How to make an architecture design, how to evaluate solution options, and the main difference between async and sync flows.
- **Surprised / broke:** The CAP concept, and that partitions are always there.
- **Open questions:** How to treat failures. *(This is the core of Modules 2–5: lost messages → Outbox (M2), duplicates and poison messages → idempotency, retries and DLQ (M3), a step failing after another succeeded → Saga and compensation (M5).)* Corrected along the way: no broker guarantees "never processed twice"; brokers deliver at least once, and the consumer must be idempotent. Weak spot to revisit: telling the root cause (a lost message) apart from the symptom (a stuck flow).
- **Next:** Module 0 Build: `docker-compose.yml` with Postgres, RabbitMQ and Redis plus health checks, then the "break it" experiment (stop a container and watch the health checks react). Then the Module 0 concept check.
- **Time:** ~5h
