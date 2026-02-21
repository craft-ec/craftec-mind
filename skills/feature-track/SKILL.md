---
name: feature-track
description: Feature-level progress tracking across components. Tracks what's really done vs what's just built. A built component is not a working feature.
user-invocable: true
trigger: "track feature", "what features are done", "feature status"; auto when declaring work complete
layer: discipline
allowed-tools: Read, Grep, Glob, Write
---

# feature-track — Feature-Level Progress Tracking

## Purpose
Stop losing progress across sessions. Stop declaring components "done" when features aren't complete. Track progress at the feature level — a feature = multiple components working together, tested end-to-end.

## Data Store

One file per craft at `.claude/features/<craft>.md`

## Protocol

### On First Use for a Craft

1. Read the design doc for the craft (`docs/<CRAFT>_DESIGN.md`)
2. Extract all specified features
3. Create `.claude/features/<craft>.md` with this format:

```markdown
---
craft: <craft-name>
type: feature
status: not-started
created: <ISO 8601 timestamp>
updated: <ISO 8601 timestamp>
tags: [<craft>, feature-tracker]
---
# <Craft> Feature Tracker

## Feature: <Feature Name>
**Design doc**: [[docs/<CRAFT>_DESIGN#<Section>]]
**Status**: ⚪ Not Started | 🟡 In Progress | 🟢 Done | 🔴 Blocked

### Components
- [ ] `<crate-name>` — <what this component does>
- [ ] `<crate-name>` — <what this component does>

### Integration Points
- [ ] <component A> ↔ <component B>: <what the connection does>
- [ ] <component C> ↔ <component D>: <what the connection does>

### End-to-End Flow
- [ ] <User action> → <step 1> → <step 2> → ... → <final result>

### Blockers
- None | <description with link to debug-log>
```

### On Completing a Component

1. Update the component's checkbox to `[x]`
2. Update the feature's `updated` timestamp
3. Check: are ALL integration points for this feature wired?
   - If not, flag what's missing: "Component X is built but not integrated with Y"
4. Do NOT change feature status to Done yet

### On Declaring a Feature "Done"

1. Verify ALL component checkboxes are checked
2. Verify ALL integration point checkboxes are checked
3. Run `integration-check` for the feature
4. Only if ALL of the above pass, set status to 🟢 Done
5. If anything fails, set status to 🟡 In Progress and document what's missing

### Status Icons

| Icon | Status | Meaning |
|------|--------|---------|
| ⚪ | Not Started | No components built |
| 🟡 | In Progress | Some components built, not all integrated |
| 🟢 | Done | All components + integration + E2E verified |
| 🔴 | Blocked | Cannot proceed; see Blockers section |

### Cross-Session Continuity

- `context-load` reads feature trackers to show what's in progress and what's next
- Session-log entries reference feature tracker for continuity
- Debug logs link back to feature blockers

## Rules

1. **Never mark a feature Done without checking integration** — a compiled component ≠ a working feature
2. **Update the tracker as you work** — not after; this prevents lost progress on context compaction
3. **One feature tracker per craft** — all features for a craft in one file
4. **Link blockers to debug logs** — `[[debug-log/<investigation>]]`
5. **Status reflects reality** — if integration is broken, status is In Progress, not Done

## Integration

- **integration-check**: Called when marking a feature Done
- **context-load**: Reads feature trackers for session context
- **audit**: Updates feature trackers as gaps are fixed
- **design-check**: Validates feature completeness against spec
