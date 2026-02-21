---
name: audit
description: Full audit-gaps workflow using orchestrator-workers pattern. Spawns parallel explorers per craft, merges findings, then fixes in batches with evaluator-optimizer loop.
user-invocable: true
disable-model-invocation: true
trigger: "find all gaps", "audit craftnet", "what's missing", "wire up everything"
layer: work
allowed-tools: Read, Grep, Glob, Bash, Write, Task
---

# audit — Comprehensive Audit Workflow

## Purpose
Find ALL gaps in the codebase and fix them systematically. Uses orchestrator-workers for discovery (parallel exploration) and evaluator-optimizer for fixing (diagnose → fix → verify → commit).

## Arguments

```
audit [scope]
```

- `all` (default): Audit entire workspace
- `<craft-name>`: Audit a specific craft (e.g., `craftnet`)
- `core`: Audit only craftec-core

## Protocol

### Phase 1: Explore (Orchestrator-Workers)

**Orchestrator**: Spawns parallel Task agents (Explore type), one per craft or area.

Each worker checks:
1. **Missing files**: Expected files that don't exist (based on crate type conventions)
2. **Cargo.toml issues**: Missing workspace inheritance, wrong dependency versions, missing members
3. **Test coverage**: Crates without test files or with empty test modules
4. **Error handling**: Missing thiserror definitions, bare `.unwrap()` in non-test code
5. **IPC completeness**: Design doc methods not wired in daemon handlers
6. **Documentation sync**: CLAUDE.md claims vs reality (listed crates that don't exist, etc.)
7. **Design doc compliance**: Code that deviates from spec (using design-check protocol)
8. **Integration gaps**: Components that compile but aren't wired together (using integration-check protocol)
9. **Dependency violations**: Sibling craft dependencies (using dep-check protocol)

**Merge findings**: Orchestrator collects all worker results and writes to `.claude/audit-gaps.md`:

```markdown
---
type: audit
status: in-progress
scope: <scope>
created: <today's date>
updated: <today's date>
tags: [audit, <scope>]
---
# Audit Gaps — <scope>

## Critical
- [ ] <description> (`<craft>/<crate>`)
- [ ] <description> (`<craft>/<crate>`)

## High
- [ ] <description> (`<craft>/<crate>`)

## Medium
- [ ] <description> (`<craft>/<crate>`)

## Low
- [ ] <description> (`<craft>/<crate>`)
```

### Phase 2: Fix (Evaluator-Optimizer)

Work through `.claude/audit-gaps.md` top-to-bottom in batches:

**Batch size**: 10-20 items

For each batch:
1. Fix the items
2. Run `verify` (evaluator)
3. If verify fails:
   - **Retry 1-2**: Read error, apply targeted fix, re-verify
   - **After 2 retries**: Switch to `debug-log` protocol (not blind retry)
4. On success: `commit-guard` per batch
5. **IMMEDIATELY mark items `[x]` in audit-gaps.md** after committing
   - This is critical — context compaction loses in-memory state
   - The file on disk is the source of truth

### Phase 3: Re-Audit

After all items are fixed:
1. Spawn fresh Explore workers to verify nothing was missed
2. If new gaps found, add them to audit-gaps.md and continue
3. **Never declare done after one pass** — cycle is: audit → fix → verify → re-audit

### Update Feature Trackers

As gaps are fixed, update `.claude/features/<craft>.md`:
- Check off components that are now complete
- Check off integration points that are now wired
- Update feature status if all components are done

## Rules

1. **Write findings to file** — NEVER hold the audit list only in memory
2. **Mark items immediately** — update checkboxes right after fixing, before moving on
3. **Batch and verify** — don't fix 100 items then verify; fix 10-20 then verify
4. **Prioritize** — Critical and High before Medium and Low
5. **Re-audit** — always do a second pass; first pass never catches everything
6. **Use protocols** — design-check and integration-check protocols, not ad-hoc checking

## Integration

- **design-check**: Workers use design-check protocol for spec compliance
- **integration-check**: Workers use integration-check protocol for wiring gaps
- **dep-check**: Workers check dependency rule compliance
- **verify**: Evaluator step after each batch
- **debug-log**: Fallback for persistent verification failures
- **commit-guard**: One commit per fixed batch
- **feature-track**: Updates feature trackers as gaps are resolved
