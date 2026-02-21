---
name: design-doc
description: Create or update design documents following Craftec conventions. Reads existing code and docs, identifies discrepancies, and ensures spec matches reality.
user-invocable: true
disable-model-invocation: true
trigger: "write a design doc", "update the spec", "document the architecture"
layer: work
allowed-tools: Read, Grep, Glob, Write
---

# design-doc — Design Document Management

## Purpose
Create and maintain design documents that serve as the source of truth for each craft's architecture, protocol, and IPC API. Design docs drive implementation (via `design-check`), not the other way around.

## Arguments

```
design-doc <craft-name|topic> [create|update]
```

- `craft-name`: The craft to document (e.g., `craftnet`, `craftobj`)
- `topic`: Or a cross-cutting topic (e.g., `settlement`, `identity`)
- `create`: Create a new design document
- `update`: Update an existing one to match current implementation

## Protocol

### For Creating a New Design Doc

1. **Read existing context**:
   - Root CLAUDE.md — ecosystem overview, shared stack, conventions
   - Craft CLAUDE.md — if it exists
   - Existing code — if any crates are already scaffolded
   - Related design docs — for consistency in style and structure

2. **Follow the standard structure**:

```markdown
# <Craft Name> Design Document

## Overview
<What this craft does, what resource it manages, key value proposition>

## Architecture
<Crate layout, component relationships, data flow diagram>

## Protocol
<Wire format, message types, state machine, handshake flow>

## Economics
<Settlement model, receipt types, pricing, epoch mechanics>

## Security
<Threat model, authentication, encryption, verification>

## IPC API
<All RPC methods with request/response types>

### `<namespace>.<method>`
**Request**: `{ field: type }`
**Response**: `{ field: type }`
**Description**: <what it does>

## Status
<Current implementation status, what's done, what's planned>
```

3. **Place the document**: `docs/<CRAFT>_DESIGN.md`

4. **Initialize feature tracker**: Create `.claude/features/<craft>.md` from the design doc's features

5. **Chain**: `commit-guard`

### For Updating an Existing Design Doc

1. **Read the existing doc** fully

2. **Read the current codebase**:
   - Scan all crates in the craft
   - Identify implemented vs. documented features
   - Find discrepancies (code that deviates from spec, spec that's not implemented)

3. **Produce a diff report**:
   ```markdown
   ## Design Doc Update: <craft>

   ### Discrepancies Found
   - ⚠️ DEVIATION: <what differs>
   - ❌ DOC ONLY: <spec'd but not implemented>
   - 🆕 CODE ONLY: <implemented but not spec'd>

   ### Proposed Updates
   1. <change to make to the doc>
   2. <change to make to the doc>
   ```

4. **Present to user for approval** — design doc changes affect all future implementation

5. **Apply approved changes**

6. **Update feature tracker** to reflect the updated spec

7. **Chain**: `commit-guard`

## Rules

1. **Design docs are prescriptive** — they define what SHOULD be built, not just what IS built
2. **Follow existing style** — match `CRAFTNET_DESIGN.md` and `CRAFTOBJ_DESIGN.md` structure
3. **IPC API is complete** — every method the daemon should support must be listed
4. **Economics section required** — even if simple, document the settlement model
5. **Security section required** — document the threat model and mitigations
6. **Update feature tracker** — when creating or updating, sync the feature tracker

## Reference Design Docs

| Doc | Craft | Quality |
|-----|-------|---------|
| `docs/CRAFTNET_DESIGN.md` | CraftNet | Good reference for protocol + IPC |
| `docs/CRAFTOBJ_DESIGN.md` | CraftObj | Most comprehensive example |
| `docs/SETTLEMENT_DESIGN.md` | Settlement | Cross-cutting design |
| `docs/CRAFTSTUDIO_DESIGN.md` | CraftStudio | Client architecture |

## Integration

- **design-check**: Design docs are the source of truth for compliance checking
- **feature-track**: Features are extracted from design docs
- **wire-ipc**: IPC methods come from the design doc's API section
- **commit-guard**: Commits design doc changes
- **context-load**: Extracts relevant design doc sections
