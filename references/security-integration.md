# Optional Cloudflare security-audit coordination

## Boundary of responsibility

This Go engineering skill **is not** a substitute for `cloudflare/security-audit-skill`. Use Cloudflare's actual installed `security-audit` skill only when the user explicitly requests a security audit or combined audit; do not invent its steps. Follow that skill's original instructions and most current installed version.

Official skill: https://github.com/cloudflare/security-audit-skill
Installation from the Cloudflare README (in your project, choose supported agent):

```bash
npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
```

Review third-party skill source before installation. Installation and execution are separate. Don't install it or download dependencies without user permission.

## Workflow when explicitly requested

1. Ensure both reviews target the **same commit and code scope** and know about uncommitted changes.
2. Keep an independent Go engineering audit and an independent Cloudflare security audit. Different agents may work in parallel only with independent output paths and no contention over the worktree. Sequential execution with independently scoped reports is also valid.
3. Honor Cloudflare's full-audit trigger and operating modes, source-grounded validation, required separate verifiers, evidence states, filesystem write isolation, and strict sandbox for target-controlled execution. If requirements aren't met, declare missing validation; do not bypass them.
4. Do not copy Cloudflare's source content into this skill or claim that manually checking OWASP items replaces its six-phase workflow.
5. After both reports are independently produced, perform a **cross-audit review of recommendations**, asking whether performance changes create new attack paths, resource amplification, side channels, or reliability failures.
6. Consolidate a deduplicated implementation plan with evidence references and separate confidence/status: security `confirmed` / `needs_validation` / `rejected` as issued by Cloudflare; Go audit `confirmed defect` / `measured bottleneck` / `code-supported risk` / `hypothesis`. Don't alter Cloudflare verdicts based on the engineering audit alone.

## Cross-audit regression checklist

- More concurrency/worker slots -> abuse-driven starvation, exhaustion, DB connection exhaustion.
- Retries and queues -> retry storms, amplification, replay abuse, DLQ poisoning, retention of PII/secrets.
- Caching -> cross-tenant leaks, invalidation, authorization changes, sensitive cache keys.
- DB query optimizations -> loss of tenant predicate, row-level authorization, consistency, transaction boundaries.
- Buffer/pointer reuse -> data leakage, mutable object reuse, races.
- Increased batching -> per-tenant fairness, rate limits, memory caps, partial failure recovery.
- Relaxed timeouts -> stuck work, service-wide resource starvation.

Output a single prioritized combined roadmap **after** independent reports. Security fixes with demonstrated concrete impact and correctness/data-loss issues outrank speculative micro-optimizations. Explicitly list incompatible recommendations and their preferred resolution.
