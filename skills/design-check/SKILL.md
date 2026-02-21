---
name: design-check
description: Verify code matches design docs before and during implementation. Stops agents from ignoring specifications and inventing their own architecture.
user-invocable: true
trigger: Auto when writing code for a designed component; "check the design", "what does the spec say"
layer: discipline
allowed-tools: Read, Grep, Glob
---

# design-check — Design Document Compliance

## Purpose
Stop agents from ignoring design documents and inventing their own architecture. Before writing or modifying code for any designed component, the specification MUST be read and followed.

## Protocol

### Step 1: Identify the Relevant Design Doc

Search for design documentation:
1. Check `docs/` for `<CRAFT>_DESIGN.md` (e.g., `docs/CRAFTNET_DESIGN.md`)
2. Check the craft's own `CLAUDE.md` for architecture notes
3. Check `docs/CRAFTEC_ECOSYSTEM.md` for cross-cutting concerns
4. Check `.claude/patterns/` for established patterns

If NO design doc exists for the component:
- **Flag this to the user**: "No design specification found for [component]. Should I check [alternative doc] or proceed with [stated assumption]?"
- Do NOT silently assume a design

### Step 2: Extract the Specification

Read the specific section describing this component. Note:
- Expected data structures and their fields/types
- Protocol/wire format definitions
- IPC method names and their request/response signatures
- Integration points with other components
- Security requirements (authentication, encryption, verification)
- Error handling expectations (what errors, how to handle them)
- Settlement/economic model if applicable

### Step 3: Compare Against Implementation

If code already exists, compare it against the spec. Produce a report:

```markdown
## Design Check: <component>
**Design doc**: <path>, section "<section name>"
**Date**: <today>

### Findings

✅ MATCHES: <what matches the spec>
⚠️ DEVIATION: <what differs from spec — describe both spec and impl>
❌ MISSING: <what the spec defines but is not implemented>
🆕 EXTRA: <what exists in code but not in spec — may be intentional>
```

### Step 4: Block on Deviations

If a deviation (⚠️) or missing item (❌) is found:
- Do NOT proceed with code changes without addressing it
- Either:
  - **Update the code** to match the spec, OR
  - **Flag the deviation** to the user with a justification for why the spec should change
- Never silently deviate from a design doc

### Step 5: For New Code

When writing new code for a designed component:
1. Read the spec section FIRST
2. Use the spec's naming, types, and structure
3. After writing, re-check against spec to verify compliance
4. Note any spec ambiguities encountered

## Design Doc Locations

| Craft | Design Doc |
|-------|-----------|
| CraftNet | `docs/CRAFTNET_DESIGN.md` |
| CraftObj | `docs/CRAFTOBJ_DESIGN.md` |
| Settlement | `docs/SETTLEMENT_DESIGN.md`, `docs/SETTLEMENT_PROGRAM_DESIGN.md` |
| Prover | `docs/PROVER_DESIGN.md` |
| Aggregator | `docs/AGGREGATOR_DESIGN.md` |
| CraftStudio | `docs/CRAFTSTUDIO_DESIGN.md` |
| Ecosystem | `docs/CRAFTEC_ECOSYSTEM.md` |

## Integration

- **context-load**: Feeds relevant design doc sections into this skill
- **feature-track**: Uses design-check results to validate feature completeness
- **wire-ipc**: Calls design-check before generating IPC methods
- **audit**: Includes design compliance in audit scope
