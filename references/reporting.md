# Evidence-based report contract

Produce an audit in this format. A short-scope audit should still use the finding categories and clearly describe gaps.

## Executive summary

- Source revision + worktree state, scope and assumptions, audited services/packages, tests/benchmarks actually executed.
- Top risk to correctness, security (only when a security review was explicitly performed), performance, reliability, database.
- Top 3–5 interventions with strongest evidence and a reason they matter.
- Explicitly state what was NOT inspected or verified.

## Findings table

| ID | Priority | Category | Status | Location | Evidence | Impact | Proposal | Risk | Verification |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

For each finding, include:

1. Repository-relative file:line and affected execution path.
2. Reproducible source evidence, safe test result, profile, metric, or plan; label unsupported assertions clearly.
3. Classification: `CONFIRMED DEFECT`, `MEASURED BOTTLENECK`, `CODE-SUPPORTED RISK`, or `HYPOTHESIS / MEASURE FIRST`.
4. Severity/priority grounded in observed or plausible impact, not checklist deviation.
5. Detailed recommendation, simpler alternatives, correctness/security implications, required regression tests, measurement plan, and rollback.

When reviewed alongside the Cloudflare audit, retain Cloudflare's own statuses and cite its verified report; **do not** reclassify `needs_validation` into confirmed.

## Prioritized implementation roadmap

- **P0:** confirmed data corruption, data loss, race/deadlock breaking service correctness, or verified severe security boundary failures.
- **P1:** measured high-impact bottlenecks, repeatable overload/availability failures, critical database latency or resource exhaustion.
- **P2:** justified medium impact improvements with well-defined validation.
- **P3:** optional/hypothesis-driven optimizations, deferred pending evidence.

For each proposed work item: affected files, dependency/order, complexity, risk, test/benchmark, target metrics and success threshold (only if grounded/agreed), rollback conditions, status `APPROVE`/`REVISE`/`MEASURE FIRST`/`REJECT`.

## Evidence log and gaps

- Commands run, environment, results, logs; identify commands not run and why.
- Database queries needing `EXPLAIN`, metrics, load characterization, or schema access.
- Potentially contradictory recommendations and plan for resolving each.

## Example concise finding

`CON-001 — CODE-SUPPORTED RISK — P1 — internal/jobs/consumer.go:86` *(example placeholder, not a real finding)*

`go func` launches one worker per message without a concurrency bound, while each worker uses the same downstream DB pool. If inflight message count is unbounded this can grow goroutines and waiting time under load. **Next step:** confirm source-visible prefetch/broker cap, queue depth, goroutine counts and pool wait. **Proposal:** only if pressure is measured, bound concurrency and set a backpressure/ack policy. **Tests:** cancellation, shutdown, duplicate processing. **Rollback:** restore previous concurrency gate and alerts.
