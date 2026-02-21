---
name: split
description: Large file refactoring guidance — analyzes files over 500 lines, identifies logical boundaries, and suggests a split plan following existing module patterns. Does NOT auto-refactor.
user-invocable: true
trigger: Auto when a file > 500 lines is read or being modified; "refactor this", "split this file"
layer: session
allowed-tools: Read, Grep, Glob
---

# split — Large File Refactoring Guide

## Purpose
When a file exceeds 500 lines, analyze it and suggest how to split it into modules following the existing codebase patterns. Presents a plan for user approval — does NOT auto-refactor.

## Protocol

### Step 1: Analyze the File

Read the file and identify:
1. **Logical sections**: Groups of related functions, impl blocks, trait implementations
2. **Section sizes**: Line count per section
3. **Dependencies**: What each section imports and exports
4. **Public API**: What's `pub` and used by other files

### Step 2: Identify Split Boundaries

Good split boundaries:
- Separate `impl` blocks for different traits
- Error types → `error.rs`
- Type definitions → `types.rs`
- Test modules → separate test files
- Configuration → `config.rs`
- Protocol/handler logic → `handler.rs` or `protocol.rs`
- Utility functions → `util.rs`

### Step 3: Check Existing Patterns

Look at how similar crates in the ecosystem are structured:
- Grep for similar crate types in the workspace
- Note how they organize modules
- Follow the established pattern

### Step 4: Present Plan

```markdown
## Split Plan: <file_path> (<line_count> lines)

### Current Structure
- Lines 1-50: Imports and types
- Lines 51-200: ServiceImpl — core logic
- Lines 201-350: ServiceImpl — IPC handlers
- Lines 351-450: Helper functions
- Lines 451-520: Tests

### Proposed Split
1. `types.rs` (50 lines) — Request/response types, enums
2. `service.rs` (150 lines) — Core service logic
3. `handler.rs` (150 lines) — IPC handler dispatch
4. `util.rs` (100 lines) — Helper functions
5. `lib.rs` (20 lines) — Module declarations and re-exports

### Migration Steps
1. Create new files
2. Move code sections
3. Update `lib.rs` with `mod` declarations and `pub use` re-exports
4. Update imports in dependent files
5. Verify with `cargo build`

### Pattern Reference
Based on: `craftnet/crates/daemon/src/` (same structure)
```

### Step 5: Wait for Approval

Do NOT proceed with the refactoring until the user approves the plan.

## Rules

1. **Never auto-refactor** — always present a plan first
2. **Follow existing patterns** — match how similar crates are structured
3. **Minimize disruption** — keep public API surface identical
4. **Re-export from lib.rs** — maintain the same import paths for consumers
5. **500 line threshold** — suggest split only for files over 500 lines

## Integration

- **verify**: After applying the split, verify compilation
- **commit-guard**: Commit the refactored modules
- **patterns**: May save the resulting module structure as a pattern
