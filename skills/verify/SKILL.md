---
name: verify
description: Build, test, clippy, and format verification pipeline with evaluator-optimizer loop. On failure, diagnoses and retries; after 2 failures, switches to structured debug-log protocol.
user-invocable: true
trigger: "check if it builds", "run tests", "verify"; auto after significant code changes
layer: work
allowed-tools: Bash
---

# verify — Build Verification Pipeline

## Purpose
Run the full Rust verification pipeline (fmt, build, clippy, test) with smart error handling. On failure, attempts diagnosis and fix. After persistent failures, escalates to the structured `debug-log` investigation protocol instead of continuing to guess.

## Protocol

### Step 1: Determine Scope

Accept optional arguments:
- `--crate <name>` or `-p <name>`: Scope to a specific crate
- `--quick`: Skip tests (only fmt + build + clippy)
- No args: Run against the full workspace

### Step 2: Run Pipeline

Execute in order, **stop on first failure**:

1. **Format check**
   ```bash
   cargo fmt --all -- --check
   ```
   - If this fails: auto-fix with `cargo fmt --all`, then continue

2. **Build**
   ```bash
   cargo build --workspace
   # or: cargo build -p <crate>
   ```

3. **Clippy**
   ```bash
   cargo clippy --workspace -- -D warnings
   # or: cargo clippy -p <crate> -- -D warnings
   ```

4. **Tests** (skip if `--quick`)
   ```bash
   cargo test --workspace
   # or: cargo test -p <crate>
   ```

### Step 3: Report Results

For each step, report:
```
✅ fmt — passed
✅ build — passed (42s)
❌ clippy — 3 warnings as errors
   → src/relay/mod.rs:45: unused variable `x`
   → src/daemon/service.rs:120: unnecessary clone
   → src/network/protocol.rs:88: unresolved import
```

### Step 4: Evaluator-Optimizer Loop (on failure)

When a step fails:

**Retry 1**: Read the error output, diagnose the issue:
- Missing import → add the import
- Type mismatch → fix the type
- Unused variable → remove or prefix with `_`
- Missing dependency → add to Cargo.toml
- Apply the fix and re-run the failed step

**Retry 2**: If the same step fails again with a different error:
- Read the new error, apply another targeted fix
- Re-run

**After 2 failed retries**: Switch to `debug-log` protocol:
- Create `.claude/debug-log/<issue>.md`
- Follow the structured investigation protocol (hypothesis → evidence → test → result)
- Do NOT continue blind retrying

### Step 5: Final Status

Report overall status:
```
## Verification: <scope>
✅ fmt
✅ build
✅ clippy
✅ tests (12 passed, 0 failed)

Overall: PASS
```

Or on failure:
```
## Verification: <scope>
✅ fmt
✅ build
❌ clippy — see debug log: .claude/debug-log/<issue>.md

Overall: FAIL — investigation started
```

## Rules

1. **Always run in order**: fmt → build → clippy → test
2. **Stop on first failure**: Don't run tests if build fails
3. **Max 2 auto-fix retries**: After that, switch to debug-log protocol
4. **Never suppress warnings**: Use `-D warnings` for clippy
5. **Report clearly**: Show exactly what passed and what failed

## Integration

- **commit-guard**: Calls verify's build step as its compile gate
- **audit**: Calls verify after each batch of fixes
- **scaffold-crate**: Calls verify after generating a new crate
- **wire-ipc**: Calls verify after wiring a new method
- **debug-log**: verify escalates to debug-log after persistent failures
