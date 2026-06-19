# Scale Plan

## Data model/indexes
- Signals table uses a primary key on `id`.
- Unique constraint on `idempotency_key` prevents duplicate signal creation.
- Composite index `(user_id, created_at)` supports efficient user signal retrieval.
- For larger datasets, partition by time or user shard.

## Idempotency across instances
- Use a database-level unique constraint on `idempotency_key`.
- Inserts are atomic and safe across multiple application instances.
- On unique constraint violation, return the existing resource instead of creating a duplicate.
- This avoids check-then-insert race conditions.

## Rate limiting across instances
- Current implementation uses in-memory counters.
- For multi-instance deployments, move rate limiting to Redis.
- Use atomic Redis INCR and EXPIRE operations for consistency across nodes.
- Optionally implement a sliding-window algorithm for smoother limiting.

## Observability (logs/metrics/alerts)
- Structured request and error logging.
- Track request rate, latency, error rate, and rate-limit violations.
- Monitor database failures and retry counts.
- Configure alerts for elevated error rates and service degradation.

## Failure modes (DB down / partial outages / retries)
- Retry transient database failures using exponential backoff with jitter.
- Return 503 when retries are exhausted.
- Preserve idempotency guarantees during retries.
- Consider circuit breakers to reduce pressure on failing dependencies.

## 10k RPS design sketch (infra & cost ballpark)
- Stateless Fastify application instances behind a load balancer.
- Redis cluster for distributed rate limiting and caching.
- PostgreSQL with connection pooling and read replicas.
- Horizontal autoscaling based on CPU and request rate.
- Queue-based processing (Kafka/SQS/RabbitMQ) for asynchronous workloads.
- Estimated deployment: load balancer, 4-8 application nodes, Redis cluster, managed database.
