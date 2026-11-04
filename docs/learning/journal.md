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

### 2026-11-03 · Module 0 · Postgres container + health checks
- **Did:** Warm-up on root cause vs symptom ("order stuck in Pending", "customer charged twice"). Covered what health checks prove (running vs listening vs ready). Wrote the Postgres service in `docker-compose.yaml` and got it `healthy`. Fixed along the way: `start_period` at service level (it belongs inside `healthcheck`), the obsolete `version` key, `postgres:latest` (now 18, which changed the data mount path) replaced by a pinned `17.11`, an unneeded `PGDATA`, and `.env.test` not being loaded (Compose only auto-loads `.env`).
- **Learned:** Pin image versions to avoid breaking changes (`latest` silently moved to a new major version and broke the mount path). Named volumes keep data across `down`; `down -v` wipes it. Compose-time substitution (`.env`, `--env-file`) is a different layer from the container's environment (`environment:`, `env_file:`). `pg_isready` proves the server accepts connections, not that a given database or user exists.
- **Surprised / broke:** The failure modes were all config validation and environment issues, not Postgres itself. `docker compose config` shows the file after variable substitution.
- **Open questions:** Corrected along the way: duplicates are normal under at-least-once delivery. Fix them at the **consumer** (idempotency), not the producer. Root causes should be concrete events ("where is the message right now?"), not design weaknesses. Still to think about (Module 3): how would Payment know it has already handled a message? Also, one Postgres user per service database (for ADR 0002)?
- **Next:**
  1. Decide where the compose file lives (root, per CLAUDE.md, or `infra/` and update CLAUDE.md) and delete the duplicate `infra/docker-compose.yaml`.
  2. Init script `infra/postgres/*.sh` (LF line endings, add `.gitattributes` with `*.sh text eol=lf`): create only the `orders`, `inventory` and `payments` databases. No tables or seed data, because services own their schemas. Bind-mount it read-only to `/docker-entrypoint-initdb.d/`. Run `down -v` first, then verify with `psql -c '\l'`.
  3. Decide what to do with `POSTGRES_DB` (`orders`, or unset and let the script create all three).
  4. Add `.env.example` with placeholder values.
  5. RabbitMQ (`-management` image, ports 5672 and 15672, `rabbitmq-diagnostics` health check) and Redis (`redis-cli ping`).
  6. "Break it": stop a container and watch `docker compose ps` and the health checks react.
  7. Module 0 concept check, then tick the roadmap boxes.
- **Time:** ~1.5h

### 2026-10-26 · Module 0 · Concepts + ADR 0001 (message broker)
- **Did:** Went through the Module 0 concepts with exercises: sync vs async, temporal and spatial coupling, commands vs events (5/5), CAP, and eventual consistency. Ran the decision-coach for the broker choice: framed the question, listed forces, compared RabbitMQ, Kafka, Redis, Azure Service Bus and Postgres-as-queue. Wrote and committed [ADR 0001: Use RabbitMQ as the message broker](../adr/0001-message-broker.md).
- **Learned:** How to make an architecture design, how to evaluate solution options, and the main difference between async and sync flows.
- **Surprised / broke:** The CAP concept, and that partitions are always there.
- **Open questions:** How to treat failures. *(This is the core of Modules 2–5: lost messages → Outbox (M2), duplicates and poison messages → idempotency, retries and DLQ (M3), a step failing after another succeeded → Saga and compensation (M5).)* Corrected along the way: no broker guarantees "never processed twice"; brokers deliver at least once, and the consumer must be idempotent. Weak spot to revisit: telling the root cause (a lost message) apart from the symptom (a stuck flow).
- **Next:** Module 0 Build: `docker-compose.yml` with Postgres, RabbitMQ and Redis plus health checks, then the "break it" experiment (stop a container and watch the health checks react). Then the Module 0 concept check.
- **Time:** ~5h
