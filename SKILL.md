---
name: go-backend-audit
description: Audit Go backends for performance, concurrency, retries, queues, and database bottlenecks. Use for Go codebase audits or optimization plans; coordinate a security audit if explicitly requested.
---

# Go backend engineering audit

## When to use

Use this skill for requests to audit, profile, optimize, or plan improvements to a Go service, especially CPU/memory, concurrency, worker pools, background jobs, retries, message delivery, and database access. Also use it to critique an existing Go optimization plan.

- **Default mode:** read-only Go engineering audit and evidence-based improvement plan.
- **Plan review mode:** review an existing plan and classify proposals as APPROVE, REVISE, MEASURE FIRST, or REJECT.
- **Combined mode:** only when the user explicitly requests a security audit as well, coordinate a separate independent audit using the installed Cloudflare `security-audit` skill (see `references/security-integration.md`). Never imply that this skill itself performs Cloudflare's full security workflow.
- **Implementation mode:** only after explicit user authorization for a specific set of changes. Re-run relevant tests and benchmarks afterward.

## Mandatory principles

1. Correctness, integrity, and safety take priority over speed. Preserve business behavior, ordering requirements, auth boundaries, and data durability.
2. Evidence first: distinguish **confirmed defect**, **measured bottleneck**, **code-supported risk**, and **hypothesis / needs measurement**. Never invent impact figures, profiles, timings, SQL plans, file locations, or test outcomes.
3. Investigate real hot paths and workloads; don't blindly recommend pointers, workers, `sync.Pool`, caching, channels, `RWMutex`, retries, DLQs, or indexes.
4. Audit is **read-only**: don't edit source, commit, deploy, alter infrastructure, or execute database migrations. Write a report only when the user explicitly requested an output artifact/location; otherwise report in the response.
5. Do not execute target-controlled code, tests, or benchmarks unless their external effects and environment are understood and safe. Never execute potentially destructive or high-load checks against production. Do not install dependencies or fetch remote packages without authorization. If safety cannot be established, provide exact commands for a sandboxed/local operator run, label results unverified.
6. Keep exact file:line references and a source revision (`git rev-parse HEAD` if available). Note dirty worktrees, missing schema files, unavailable metrics, and unrun commands.
7. Respect user scope. For broad audits, prioritize coverage of entrypoints, hot paths, workers, persistence, and shutdown rather than claiming exhaustive validation from a single pass.

## Procedure

### 1. Scope and architecture

- Determine Go version, executable entrypoints, modules, critical request/job flows, dependencies, external systems, datastore(s), background workers, deployment/traffic assumptions, tests, metrics, and migrations.
- Trace data flow and resource ownership: goroutines, contexts, locks, channels, queues, DB transactions, message acknowledgements, network clients, and shutdown.
- Record missing workload assumptions rather than silently choosing them. If an optimization plan is supplied, map every proposal to code evidence.

### 2. Focused review

Read the following references, focusing on relevant subsystems:

- [`references/performance.md`](references/performance.md) — copies vs pointers, allocations, algorithms, profiles, batching.
- [`references/concurrency.md`](references/concurrency.md) — races, channels, shutdown, worker pools, bounded parallelism.
- [`references/reliability.md`](references/reliability.md) — retries, acknowledgements, DLQ, transactional boundaries, idempotency.
- [`references/database.md`](references/database.md) — SQL, indexes, query plans, pools, transaction scope.

Classify issue severity by impact, not checklist deviation. For suspected defects, seek a minimal source-grounded example or a safe bounded test.

### 3. Safe verification

When tools and environment are available and safe, consider the following commands, never treating any single command as proof of safety:

```bash
go version
git status --short
go vet ./...
staticcheck ./...               # only if installed
go test ./...                  # only if tests have safe effects
go test -race ./...            # exercises paths tests actually execute
go test -race -count=5 ./...   # targeted packages preferred for costly suites
go test -run='^$' -bench=. -benchmem ./path/to/pkg  # targeted benchmarks
```

- Avoid running potentially side-effectful tests or fetching dependencies just to complete an audit.
- Compare before/after latency (p50/p95/p99), throughput, CPU, allocations (`B/op`, `allocs/op`), heap/GC, goroutine counts, lock/block profiles, queue wait/lag, retry and DLQ metrics, and DB latency/rows examined/pool waits.
- Escalate to `pprof`, tracing, or query plans only with an appropriate safely isolated environment. PostgreSQL `EXPLAIN ANALYZE` **executes** the query; don't run it on production or mutating statements without explicit safe authorization.
- Static review cannot prove absence of races, deadlocks, or resource leaks. A race detector only sees execution paths and schedules exercised during tests.

### 4. Challenge the plan

For **every** proposal ask: What evidence? Why now? Could simpler code work? What invariant changes? Failure/cancellation/overload/shutdown behavior? Security and tenant-isolation impact? DB/downstream amplification? Validation and rollback? Categorize APPROVE, REVISE, MEASURE FIRST, or REJECT.

### 5. Deliver

Use [`references/reporting.md`](references/reporting.md) for the report contract: summary, findings with evidence and file:line, missing data, safe verification, before/after metrics, prioritized P0-P3 roadmap, accepted/rejected proposals, and rollback checks.

Do not equate "performance best practice" with measured impact. Propose changes, not implementation, unless explicitly approved.

### 6. Security integration, if requested

Read [`references/security-integration.md`](references/security-integration.md). Keep security and engineering findings independent until a separate cross-audit review. If the Cloudflare skill isn't installed or mandatory isolation controls are absent, report the blocker and a setup path, rather than improvising or claiming the full Cloudflare audit ran.
