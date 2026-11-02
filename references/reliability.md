# Reliability, retries, messaging, and operations

## Inspect

- Define delivery guarantee and state transitions for each queue or job (at-most-once, at-least-once, ordering, redelivery, idempotency).
- Trace broker ack/nack timing versus database commits and external side effects. Identify at-risk crash windows, missing deduplication, and poison messages.
- Validate bounded retries: retryable vs permanent errors, exponential backoff, jitter, max attempts / elapsed time, request deadlines, per-attempt timeouts, and cancellation.
- Analyze aggregate fan-out/retry amplification across layers; check thundering herds and upstream/downstream quota exhaustion.
- Evaluate DLQ **when applicable to durable asynchronous work**; define ownership, metadata/redaction, message TTL/retention, alerting, replay policy, idempotency on replay, and how operators recover safely. A DLQ alone does not fix a poison-message bug.
- Analyze partial failures and durable effects across DB and message broker; consider outbox/inbox patterns only where evidence requires cross-system guarantees.
- Check error wrapping/classification, deadlines, graceful shutdown, inflight draining, visibility into queue depth/lag, attempt counts, success/failure latency, and DLQ age.
- Check circuit breakers/bulkheads only with clear failure domain and operational cost; guard against fail-open behavior and unintentionally amplified traffic.

## Never automatically

- Retry all errors; retry cancellation, authorization failures, invalid requests, or known permanent failures.
- Add DLQs to local in-process channels or ephemeral tasks without a durability requirement.
- Promise exactly-once processing merely because a broker has ack/retry support.
- Recommend concurrency increases without checking queues, database pools, and downstream limits.

## Proof/validation

Record the exact failure state, crash window, possible duplicate/loss, the user-visible and business impact, bounded local reproduction, metrics to track, and how to rollback. Test transient failure, permanent failure, duplicate delivery, cancellation, delayed ack, full queue, and restart/replay.
