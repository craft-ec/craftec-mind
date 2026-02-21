---
name: dep-check
description: Verify the dependency rule invariant — every craft depends ONLY on craftec-core, never on sibling crafts. Auto-runs after Cargo.toml edits.
user-invocable: true
trigger: "check dependencies"; auto after Cargo.toml edits
layer: work
allowed-tools: Bash, Read, Grep, Glob
---

# dep-check — Dependency Rule Validation

## Purpose
Enforce the Craftec ecosystem's core architectural invariant: every craft depends **only** on `craftec-core` — never on sibling crafts. CraftStudio is the sole exception (it may depend on any craft it ships with).

## The Rule

```
craftnet    → craftec-core ✅
craftobj    → craftec-core ✅
craftnet    → craftobj     ❌ VIOLATION
craftstudio → craftnet     ✅ (exception: craftstudio can depend on any craft)
```

## Protocol

### Step 1: Enumerate All Crafts

Find all craft directories (submodules) in the workspace:
- Glob for `*/Cargo.toml` at the workspace root
- Identify which are "crafts" (craftnet, craftobj, craftsec, craftsql, craftvfs, cloakcraft, etc.)
- Note which is CraftStudio (exempt from the rule)

### Step 2: Check Each Craft's Dependencies

For each craft (excluding craftstudio):
1. Find all `Cargo.toml` files: `<craft>/**/Cargo.toml`
2. Read the `[dependencies]` and `[dev-dependencies]` sections
3. Check for references to sibling crafts:
   - Direct: `craftnet = { path = "..." }` or `craftnet = "..."`
   - Via craftec-core is fine: `craftec-core = ...`, `craftec-crypto = ...`, etc.
   - Workspace inheritance: `<sibling>.workspace = true`

### Step 3: Check Workspace Consistency

Verify the root `Cargo.toml`:
- All workspace members are listed
- Workspace dependencies are consistent (no duplicate versions)
- No circular dependencies

### Step 4: Report

```markdown
## Dependency Check

### Violations
❌ craftobj/crates/daemon/Cargo.toml:15 — depends on `craftnet` (sibling dependency)

### Warnings
⚠️ craftnet/crates/network/Cargo.toml:22 — `craftec-network` version mismatch with workspace

### Passed
✅ craftnet — clean (depends only on craftec-core crates)
✅ craftsec — clean
✅ craftstudio — exempt (may depend on any craft)

Overall: FAIL (1 violation)
```

### Step 5: On Violation

- Report the exact file path and line number
- Suggest the fix (usually: depend on the shared craftec-core crate instead)
- Do NOT auto-fix dependency violations — they may indicate an architectural issue that needs discussion

## Rules

1. **CraftStudio exception** — craftstudio may depend on any craft
2. **craftec-core crates are always OK** — any craft may depend on any crate under `craftec-core/crates/`
3. **Third-party crates are fine** — only sibling CRAFT dependencies are violations
4. **Report, don't auto-fix** — dependency violations may be architectural decisions

## Integration

- **scaffold-crate**: Runs dep-check after creating a new crate
- **new-craft**: Runs dep-check after scaffolding a new craft
- **commit-guard**: May be called before committing Cargo.toml changes
- **audit**: Includes dep-check in audit scope
