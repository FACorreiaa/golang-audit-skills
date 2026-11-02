# Go concurrency review

## Trace lifecycle

- Inventory goroutine creation sites, start conditions, lifetime ownership, cancellation, shutdown, join/wait, error propagation, and limiting mechanism.
- For channels document **who sends, who receives, who closes, and why closing is needed**. Only a sender with exclusive knowledge that all sends are complete should close a channel; many channels need not be closed. Detect double close, send-on-closed-channel panics, receiver/sender deadlocks, and indefinite blocking.
- Trace `context.Context` from request/job entry to DB/network calls. Confirm deadline/cancellation actually releases blocked work; don't store request contexts for unrelated long-lived work.
- Check synchronization: map access, shared slices, struct fields, races, locks held through I/O, copied mutexes, lock order, unnecessary `RWMutex`, atomic invariants, check-then-act bugs, and ownership/immutability.
- Check `WaitGroup` use, group cancellation, panic handling, and worker shutdown. A WaitGroup alone cannot stop blocked work.

## Parallel-processing decisions

- Establish whether tasks are CPU-bound, I/O-bound, dependent, or ordering-sensitive.
- Identify unbounded goroutine spawning, uncontrolled batch fan-out, and unbounded in-memory queues.
- Consider worker pools or semaphores **when bounded concurrency is needed**; size against actual CPU, downstream quotas, database pool size, and fairness. Concurrency may increase throughput until saturation, then degrade latency.
- Require backpressure and overload policy: block/cancel/reject/drop/durable queue. Identify which semantics are acceptable.
- Check queue capacity, head-of-line blocking, starvation, work ownership, duplicates, and stale work during shutdown.
- Prefer sequential execution when it is simpler and sufficiently fast.

## Evidence and tests

- Use `go test -race` for exercised paths, repeated targeted tests for intermittent bugs, deterministic cancellation/shutdown tests, bounded mock work, and goroutine/lock/block profiles.
- Probe behavior for no consumers, slow consumers, cancellation mid-send, downstream timeout, panic/error, premature close, and shutdown with pending work.
- Don't claim the race detector proves no logical races, deadlocks, or leaks; distinguish data races from ordering/transactional races.
