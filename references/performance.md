# Go performance inspection

## Investigate

- Find top CPU / alloc sources in measured hot paths; choose algorithmic and I/O wins before micro-optimizations.
- Look for repeated expensive parsing, JSON marshalling, reflection, conversions, regex compilation, and redundant computation.
- Look for avoidable struct/slice/map copying, repeated allocations in loops, poor capacity sizing, unbounded buffers, and unnecessary temporary objects.
- Compare large values vs pointers *case by case*: copying large immutable structs may cost CPU, but pointers can add aliasing, cache misses, escaping allocations, and races. Do not recommend a pointer change without inspecting mutability, receiver semantics, and profiling/benchmarking.
- Assess `strings.Builder`, `bytes.Buffer`, `sync.Pool`, pooling of clients/resources, batching, streaming, and reusable buffers only when the lifetime and ownership are explicit.
- Check slice retention (small subslices retaining large arrays), unsafe buffer reuse, hidden allocations, and GC behavior.
- Inspect serialization/transport overhead, network round trips, request fan-out, timeout budgets, and redundant work. Review benchmark representativeness before recommending changes.

## Don't

- Don't universally prefer pointer receivers/arguments, `sync.Pool`, zero-copy tricks, or manual memory tricks.
- Don't introduce unsafe code or shared mutable caches solely to improve microbenchmarks.
- Don't claim lower allocation counts imply lower p99 latency or better throughput.
- Don't optimize cold paths without demonstrated payoff.

## Proof required

- Exact code path and realistic workload.
- Baseline/profile or clearly labeled estimate/hypothesis, with environment and sample conditions.
- Validation proposal: scoped benchmark, pprof CPU/heap, allocs/op, application p95/p99, load test in an isolated environment, regression tests.
- Discuss memory/correctness tradeoffs and rollback criteria.
