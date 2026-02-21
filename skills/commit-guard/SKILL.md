---
name: commit-guard
description: Git commit guardian — the single exit point for all commits. Verifies format and compilation before staging, generates context-aware commit messages, never stages secrets.
user-invocable: true
trigger: "commit this", "save changes"; auto-chained from other skills after completing work
layer: guardian
allowed-tools: Bash, Read, Grep, Glob
---

# commit-guard — Git Commit Guardian

## Purpose
The single exit point for all git commits in the Craftec ecosystem. Every skill that produces changes chains into commit-guard. It ensures quality gates pass before any commit.

## Protocol

### Step 1: Format Gate

```bash
cargo fmt --all -- --check
```

If formatting fails:
- Run `cargo fmt --all` to fix
- Continue to next step (format issues are auto-fixable)

### Step 2: Compile Gate

```bash
cargo build --workspace
```

Or, if the changes are scoped to specific crates:
```bash
cargo build -p <changed-crate-1> -p <changed-crate-2>
```

If compilation fails:
- Do NOT commit
- Report the error
- The calling skill should fix the issue and re-invoke commit-guard

### Step 3: Stage Relevant Files

**NEVER use `git add -A` or `git add .`**

Stage only the files that were intentionally changed:
- Use `git add <specific-file>` for each changed file
- Review `git status` to identify what changed
- **NEVER stage**: `.env`, credentials, secrets, `target/`, `.DS_Store`
- Verify staged files with `git diff --cached --stat`

### Step 4: Generate Commit Message

Format: `<type>: <description>`

Types:
- `feat`: New feature or capability
- `fix`: Bug fix
- `refactor`: Code restructuring without behavior change
- `chore`: Maintenance, dependency updates, config changes
- `docs`: Documentation changes
- `test`: Adding or updating tests

Rules:
- **No signatures** — no `Signed-off-by`, no `Co-Authored-By` (per global CLAUDE.md)
- **No empty commits** — if nothing to stage, report and exit
- Description should be concise (under 72 chars) and describe the WHY, not just the WHAT
- When chained from another skill, the message should reflect what that skill did:
  - From `audit`: `fix: resolve <N> audit gaps in <craft> (<summary>)`
  - From `scaffold-crate`: `feat: scaffold <crate-name> crate for <craft>`
  - From `wire-ipc`: `feat: wire <method-name> IPC method in <craft>`
  - From `design-doc`: `docs: create/update <craft> design document`
  - From `patterns`/`session-log`: `chore: update knowledge base`

### Step 5: Commit

```bash
git commit -m "<type>: <description>"
```

### Step 6: Verify

```bash
git status
```

Confirm the working tree is clean (or only has expected unstaged files).

## Context-Aware Behavior

When chained from other skills, commit-guard receives context about what was done:
- It does NOT re-run full verification if the calling skill already ran `verify`
- It DOES always run `cargo fmt --check` and `cargo build` as final gates
- The commit message reflects the calling skill's work

## Rules

1. **Never skip format/compile gates** — even for "trivial" changes
2. **Never stage secrets** — check for `.env`, `*credentials*`, `*secret*`, `*key*` patterns
3. **Never use `git add -A`** — always stage specific files
4. **One commit per logical unit** — don't batch unrelated changes
5. **Clean messages** — no signatures, no attribution, no emoji

## Integration

- All other skills chain into commit-guard as their final step
- `verify` is called BY commit-guard (compile gate), not the other way around
- `dep-check` may be called before commit-guard when Cargo.toml files are staged
