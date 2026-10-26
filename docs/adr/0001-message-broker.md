# 0001. Use RabbitMQ as the message broker

- Status: accepted
- Date: 2026-10-26

## Context
What message broker use? Take account on Reliable messaging it should guarante every message is delivered at least once, so duplicates are possible and consumers must be idempotent. Dlq it would be filled with messages that achieve the number of retries allowed, simplicity and understand before abstacting and also the main purpouse of learning from project, once the project will run locally and we not have a lot of time to implement it, we will use a broker that is robust enough to teach but simple enough to be used on time.

## Options considered
1. **RabbitMQ**: It has most of the needed for project built in / It delete message after proccess
2. **Kafka**: It has huge log for messages / It is a bit larger and It will be necessary to implement some of the needs like Dlq
3. **Redis**: It has two different services that can be used for the project the pub/sub and the streams / Temporal coupling is back on pub/sub and streams tech different concepts from this project goal
4. **Azure Service Bus**: It is robust / it breaks runs locally force, and you can't run the break-it experiments against it
5. **Postgres as a queue**: It would be implemented in a database that is already in use / All the proccess logic would be implemented manually

## Decision
We choose **RabbitMq** because it has exchanges to be used one queue for service, it also has retry and built in and will be simpler to implement it.

## Consequences
It will be easier to implement constitution and forces listed, it will be challange to implement the old messages processing once we will not have log on it, it will be necessary to implement on order service a way to resend old orders, and if it was the wrong choice I will be able to apply the learned concepts across any other choice, but I will need to implement the dlq manually. 

## Follow-ups
- Kafka may be added in Module 7 (optional), through its own ADR (0008), alongside RabbitMQ rather than replacing it. That ADR should revisit what we gave up here: keeping and replaying old messages.
- This ADR records the reasoning behind the constitution's "RabbitMQ is the primary message broker" (Technology & Architecture Constraints).

