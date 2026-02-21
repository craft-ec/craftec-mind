---
name: new-craft
description: Scaffold an entire new craft project — creates all standard crates, CLAUDE.md, updates ecosystem docs, initializes feature tracker. Uses orchestrator-workers for parallel crate scaffolding.
user-invocable: true
disable-model-invocation: true
trigger: "spin up a new craft", "create a new project"
layer: work
allowed-tools: Read, Grep, Glob, Bash, Write, Task
---

# new-craft — New Craft Project Scaffolding

## Purpose
Create a complete new craft project following the Craftec ecosystem conventions. Scaffolds all standard crates, creates documentation, updates the ecosystem, and initializes tracking.

## Arguments

```
new-craft <craft-name> <resource-type> <description>
```

- `craft-name`: Name of the new craft (e.g., `craftmsg`)
- `resource-type`: What resource this craft manages (e.g., `messaging`, `storage`)
- `description`: Brief description of the craft's purpose

## Protocol

### Step 1: Validate

1. Check that the craft name doesn't conflict with existing crafts
2. Verify it follows naming conventions (`craft<suffix>` or `<prefix>craft`)
3. Confirm with user before proceeding

### Step 2: Create Directory Structure

```
<craft-name>/
├── Cargo.toml          (workspace root for the craft)
├── CLAUDE.md           (craft-level instructions)
└── crates/
    ├── core/           (types, traits, errors)
    ├── daemon/         (service, IPC handler)
    ├── network/        (libp2p protocol, optional)
    └── cli/            (CLI tool, optional)
```

### Step 3: Scaffold Crates (Orchestrator-Workers)

Spawn parallel workers to scaffold independent crates:
- Worker 1: `scaffold-crate <craft> core`
- Worker 2: `scaffold-crate <craft> daemon`
- Worker 3: `scaffold-crate <craft> network` (if applicable)
- Worker 4: `scaffold-crate <craft> cli` (if applicable)

Each worker follows the `scaffold-crate` protocol.

### Step 4: Create Craft CLAUDE.md

```markdown
# <Craft Name>

<description>

## Architecture

| Crate | Purpose |
|-------|---------|
| `<craft>-core` | Types, traits, errors |
| `<craft>-daemon` | Service daemon with IPC |
| `<craft>-network` | P2P protocol (if applicable) |
| `<craft>-cli` | CLI interface (if applicable) |

## Development Status
- [x] Crate structure scaffolded
- [ ] Core types defined
- [ ] Daemon service implemented
- [ ] IPC methods wired
- [ ] Network protocol implemented
- [ ] CLI tool implemented
- [ ] Tests written
- [ ] Integration verified

## Dependency Rule
This craft depends ONLY on `craftec-core`. Never add dependencies on sibling crafts.
```

### Step 5: Update Ecosystem

1. Update root `CLAUDE.md` ecosystem table with the new craft
2. Add the craft to the workspace if using a root workspace

### Step 6: Initialize Feature Tracker

Create `.claude/features/<craft>.md` with features pre-populated from the description:
- Core types and traits
- Daemon service
- IPC methods (from design doc if available)
- Network protocol (if applicable)

### Step 7: Run Validations

1. `dep-check` — verify no dependency violations
2. `verify` — confirm everything compiles

### Step 8: Commit

Chain into `commit-guard` with message: `feat: scaffold <craft-name> project (<resource-type>)`

## Rules

1. **Design doc first** — if a design doc exists for this craft, read it before scaffolding
2. **Dependency rule** — new craft depends ONLY on craftec-core
3. **Minimal scaffolding** — create compilable stubs, not feature-complete code
4. **Initialize tracking** — always create feature tracker
5. **Update ecosystem docs** — root CLAUDE.md must reflect the new craft

## Integration

- **scaffold-crate**: Used as worker for each crate
- **dep-check**: Validates after scaffolding
- **verify**: Confirms compilation
- **commit-guard**: Final commit
- **feature-track**: Initializes feature tracker
- **design-doc**: May be used to create the design doc
