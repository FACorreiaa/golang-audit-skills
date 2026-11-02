# go-backend-audit Agent Skill

A reusable, evidence-first Go backend engineering audit: performance, concurrency, reliability, queues/retries/DLQ, and databases. Optional coordinated review with Cloudflare's `security-audit` skill when explicitly requested.

## Install in a repository

**Codex and Cursor** (both recognize the common `.agents/skills` location):

```bash
mkdir -p .agents/skills
cp -R go-backend-audit .agents/skills/
```

**Claude Code**:

```bash
mkdir -p .claude/skills
cp -R go-backend-audit .claude/skills/
```

**Personal use across repositories**:

```bash
mkdir -p ~/.agents/skills
cp -R go-backend-audit ~/.agents/skills/
# For Claude Code instead: mkdir -p ~/.claude/skills && cp -R go-backend-audit ~/.claude/skills/
```

These commands assume the `go-backend-audit/` folder is available in your shell's current directory (e.g. after unzipping this archive). For project installation, run from the target repository root. For remote agents, ensure the skill folder exists in the remote workspace or is checked into the repository.

## Invoke

- **Codex**: `$go-backend-audit Audit this repository and propose a read-only improvement plan.`
- **Claude Code**: `/go-backend-audit Audit this backend (no code changes).`
- **Cursor**: `/go-backend-audit Audit this backend (no code changes).`

Auto-invocation is also possible when your prompt matches the skill description, but explicit invocation is more dependable for critical reviews.

For **combined** review, install Cloudflare's independent security skill separately and explicitly request both:

```bash
npx skills add https://github.com/cloudflare/security-audit-skill --skill security-audit
```

Then: `Use go-backend-audit and security-audit independently on the same commit, and perform a cross-audit review. Do not modify code.`

## Notes

- The skill is instruction-only. It does not run a scanner by itself; the agent must have repository access and permitted tools to inspect/run code.
- Review security of third-party skills before installing.
- Never run destructive or production-affecting tests/benchmarks without an appropriate approved environment.
- An absent security skill must be reported as absent, not silently replaced with a superficial security checklist.
# golang-audit-skills
