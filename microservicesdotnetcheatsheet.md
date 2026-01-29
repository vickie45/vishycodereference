# Microservices (.NET) Cheatsheet — Quick Learning & Revision

Goal: concise microservices architecture patterns, .NET implementations, and best practices.

## Boundaries & data
- Services per business capability; avoid shared DBs.
- Use contracts (OpenAPI) and versioning.

## Communication
- Synchronous: HTTP/REST, gRPC
- Asynchronous: Kafka, RabbitMQ for resilience and decoupling

## Data consistency
- Database per service; eventual consistency and sagas for distributed transactions.
- Outbox pattern to reliably publish events after DB commit.

## Resilience patterns
- Retry, Circuit Breaker, Bulkhead (Polly)
- API Gateway for routing and auth

## Observability
- Structured logs (Serilog), Traces (OpenTelemetry), Metrics (Prometheus)
- Health checks and readiness/liveness probes

## Deployment & scaling
- Containerize, deploy to Kubernetes, use probes and autoscaling.
- CI/CD: build, test, push images, deploy.

## Security
- OAuth2/OpenID Connect for auth, secure secrets (Key Vault), mTLS for service-to-service.

## Practical patterns & operational guidance

### Outbox pattern (reliable messaging)
- Write business data and outbox row in same DB transaction. Background publisher reads outbox and publishes to broker, marks event as sent.
- Use DB index on outbox status and batch processing to avoid contention.

### Sagas for distributed transactions
- Orchestrator pattern: central saga orchestrates steps and compensations.
- Choreography: services emit events and react; simpler but harder to visualize.

### Idempotency & deduplication
- Important for retries: use idempotency keys stored in DB or idempotent consumer logic (dedup by event id).

### API versioning & backward compatibility
- Use URL versioning (/v1/...) or header versioning; maintain backward-compatible changes and provide migration window.
- Contract tests (Pact) to verify provider/consumer compatibility.

### Schema migration strategy
- Backwards-compatible schema changes: add columns nullable, deploy consumers that tolerate both, backfill, then remove old code.
- Use feature flags to toggle new behavior.

### Observability & runbook
- Standardize logs (correlation id), metrics (latency, error rate), and traces.
- Maintain simple runbooks for common incidents (db down, broker lag, high CPU).

### Deployment practices
- Canary or blue/green deploys for critical services.
- Use Kubernetes readiness/liveness probes and limit resource requests/limits.

## Real-world snippet — Outbox
- Insert domain data and outbox row in same DB transaction; background publisher reads outbox and publishes to broker.
