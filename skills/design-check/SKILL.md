---
name: design-check
description: Verify code matches design docs before and during implementation. Stops agents from ignoring specifications and inventing their own architecture. All reviews are recorded to the persistent log.
user-invocable: true
trigger: Auto when writing code for a designed component; "check the design", "what does the spec say"
layer: discipline
allowed-tools: Read, Grep, Glob, Write, Edit
---

# design-check — Design Document Compliance

## Purpose
Stop agents from ignoring design documents and inventing their own architecture. Before writing or modifying code for any designed component, the specification MUST be read and followed. **Every review MUST be recorded.**

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
**Timestamp**: <ISO 8601 full timestamp, e.g. 2026-02-21T14:30:00Z>

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

### Step 6: Record the Review (MANDATORY)

**Every design check MUST be recorded to a per-session file.** This is not optional — unrecorded reviews are wasted work that future agents cannot reference.

**Directory**: `.claude/design-reviews/`

**File naming**: One file per session, timestamped: `.claude/design-reviews/<YYYY-MM-DDTHHMMSS>Z.md`
Examples:
- `.claude/design-reviews/2026-02-21T183000Z.md`
- `.claude/design-reviews/2026-02-22T091500Z.md`

Use the timestamp of the **first review** in the session as the filename. If multiple components are reviewed in one session, they all go in the same file.

**Format**: Each session file contains Obsidian-compatible frontmatter and all reviews performed during that session:

```markdown
---
type: design-review
timestamp: 2026-02-21T14:30:00Z
components: [craftstudio-ipc, craftnet-daemon]
verdicts: [PASS_WITH_NOTES, PASS]
tags: [design-review, craftstudio, craftnet, ipc]
---

# Design Review Session: 2026-02-21T14:30:00Z

**Context**: <brief description of what work triggered these reviews>

---

## <component name>
**Timestamp**: 2026-02-21T14:30:00Z
**Design doc**: <path>, section "<section name>"
**Trigger**: <what triggered this check — e.g. "pre-implementation", "post-implementation", "user-invoked", "auto during code change">
**Verdict**: <PASS | PASS_WITH_NOTES | DEVIATIONS_FOUND | BLOCKED>

### Findings

✅ MATCHES:
- <item>

⚠️ DEVIATIONS:
- <item: spec says X, impl does Y>

❌ MISSING:
- <item>

🆕 EXTRA:
- <item>

### Disposition
- <For each deviation/missing: "ACCEPTED: reason" | "FIX NEEDED: description" | "SPEC UPDATE NEEDED: description">

---

## <next component, if multiple reviewed in same session>
...
```

### Frontmatter fields

| Field | Description |
|-------|-------------|
| `type` | Always `design-review` |
| `timestamp` | ISO 8601, matches filename |
| `components` | List of components reviewed in this session |
| `verdicts` | List of verdicts, same order as components |
| `tags` | Obsidian tags: always includes `design-review`, plus craft names, component areas |

**Why per-session files**:
- Clean timeline of all design reviews across the project
- Full context of what was checked together in one session
- Each file is immutable after the session — no append races
- Directory listing sorted chronologically shows review cadence
- Easy to correlate reviews with commits made in the same session

**Why this matters**: Without recording, each session rediscovers the same gaps. The log enables:
- Tracking whether deviations are intentional or accidental
- Knowing when a component was last verified
- Detecting spec drift over time
- Preventing duplicate reviews of the same component

### Step 7: Extract TODOs (MANDATORY)

After recording the review, extract all **FIX NEEDED** and **SPEC UPDATE NEEDED** items from the Disposition section and add them to `.claude/todos.md` using the `todo-track` skill format. This ensures action items are not buried in review documents.

For each actionable disposition item, create a TODO entry with:
- Source: link to the review file
- Component: which component is affected
- Priority: `high` for FIX NEEDED, `medium` for SPEC UPDATE NEEDED

See the `todo-track` skill for the full TODO format.

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
- **todo-track**: Receives FIX NEEDED / SPEC UPDATE NEEDED items from design-check dispositions
