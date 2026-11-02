# Database performance and correctness

## Inventory

- Identify engine and versions, schema/migrations, existing indexes/constraints, ORM, query builders, raw SQL, materialized views, and data size/cardinality if observable.
- Map concrete queries to request/job paths. Search for N+1 calls, repeated query loops, unbounded fetches, expensive JOIN/GROUP BY/ORDER BY, overfetching columns, and excessive round trips.
- Review filtering and pagination: offsets on large datasets vs keyset pagination when compatible with product semantics; stable ordering and tie-breakers.
- Check transaction length, lock behavior, consistency/isolation guarantees, read-modify-write races, atomic updates, and retry semantics around deadlocks/serialization failures.
- Inspect connection pool settings and pool waits relative to service concurrency and DB capacity. Ensure rows, statements, and transactions are closed/committed/rolled back on all paths.
- Verify `context.Context` deadlines reach database operations. Consider batching and bulk writes with transactional and memory tradeoffs.

## Index assessment

- For every candidate, cite an exact query (or query template), relevant table and predicates, JOIN, sort, existing index, and expected cardinalities/selectivity. Consider equality, range, sort, partial/covering indexes, and multi-column ordering as engine-dependent tradeoffs.
- Verify with available safe explain plans/statistics and production **read-only observability**, where authorized. A sequential scan is not automatically bad; composite column order is workload-dependent.
- Include write amplification, storage, maintenance overhead, lock implications, and rollout for each added index. Consider redundant indexes before suggesting new ones.
- Don't execute migrations or heavy/mutating `EXPLAIN ANALYZE` against production. If stats are unavailable, label the proposed index as `MEASURE FIRST`.

## Security and data isolation

Never change SQL/ORM code in a way that drops authorization checks, tenant filters, soft-delete rules, row-level security assumptions, or consistency invariants. Cross-check caching keys and batched reads against tenant boundaries.

## Deliverable

For each database finding: exact source location, query shape, schema/index evidence, likely bottleneck, explain-plan evidence (or missing data), proposed option(s), impact of the change on reads and writes, safe validation plan, and rollback approach.
