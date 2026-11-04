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

### 2026-11-04 · Module 0 · Compose complete, break-it experiments, concept check ✅
- **Did:** Finished `infra/docker-compose.yaml`. Postgres: `init.sh` creates `orders`, `inventory` and `payments` from `POSTGRES_MULTIPLE_DATABASES`, mounted read-only into `/docker-entrypoint-initdb.d`. RabbitMQ: pinned `4.x-management` image, fixed `hostname`, volume, `rabbitmq-diagnostics ping`. Redis: AOF persistence, `--requirepass`, `REDISCLI_AUTH` with `ping | grep -q PONG`. Added `infra/.env.example`, updated the CLAUDE.md commands, committed (`89fb748`). Ran the "break it" experiments. Module 0 concept check: **7.25/8 (91%)**.
- **Learned:** Init scripts only run on an empty volume (bootstrap); changes after that need migrations or manual SQL. Inside a container, `localhost` is the container itself, so use the service name. Shell variables override `.env` (`${USER}` gave my WSL username), so always prefix variables. RabbitMQ stores data per node name (`rabbit@<hostname>`), so it needs a stable hostname; Postgres uses a fixed path. Stable identity matters (message IDs, correlation IDs later). Settings passed on every start (Redis password) differ from settings written into the data on first boot (Postgres and RabbitMQ credentials).
- **Surprised / broke:**
  - `docker compose kill` did **not** trigger `restart: always`. Docker cares who stopped the container, not how (exit 137 = SIGKILL). A crash from inside (`rabbitmqctl stop`) restarted in ~1 s.
  - `pause` froze the health checks too: the status stayed `Paused` and never went `unhealthy`. No alert ≠ healthy.
  - Without `hostname`, a RabbitMQ queue vanished after `down`/`up` even with a volume.
  - The bash `for` loop inside a SQL heredoc, and connecting with `--dbname` to the database I was creating.
- **Open questions:** Concept check weak spot (Q7): when a dependency hangs, Docker won't restart it and restarting the caller doesn't help. The caller needs timeouts and to fail fast. Still open for Module 3: where should Payment record "already handled message X" so it can't drift from the charge itself (Redis persistence only narrows the window)?
- **Next:** Module 1, place order the naive way. Learn service boundaries, one database per service and the **dual-write problem**. ADR 0002 (service split and data ownership, including one Postgres user per service). Then `/speckit-specify` for `001-place-order`.
- **Time:** ~2.5h

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
