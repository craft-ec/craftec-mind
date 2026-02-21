---
name: context-load
description: Smart context loader — extracts relevant sections from design docs, feature trackers, debug logs, and session history when starting work on a craft. Uses hybrid search, not full doc dumps.
user-invocable: false
trigger: Auto when starting work on a specific craft
layer: session
allowed-tools: Read, Grep, Glob
---

# context-load — Smart Context Extraction

## Purpose
Load relevant context before working on a craft. Does NOT dump entire documents — extracts only the sections relevant to the current task. Uses hybrid search (keyword grep + semantic reading).

## Protocol

### Step 1: Identify the Craft

Determine which craft is being worked on from the conversation context:
- Explicit mention: "work on craftnet", "fix the relay"
- Implicit: file paths being edited contain a craft name
- If unclear, do not load — wait for clarity

### Step 2: Extract Context (Smart, Not Full Dump)

Load these sources, extracting only relevant sections:

#### Root CLAUDE.md
- The craft's row in the ecosystem table
- The dependency rule
- IPC conventions section
- Relevant shared technology notes

#### Craft CLAUDE.md
- Architecture overview
- Current crate layout
- Development status

#### Design Doc (`docs/<CRAFT>_DESIGN.md`)
- **Only sections relevant to the current task**
- Use Grep to find the section, then Read with offset/limit
- Prepend each section with: "From [doc], section [X], describing [Y]:"

#### Feature Tracker (`.claude/features/<craft>.md`)
- Current feature status — what's in progress, what's blocked
- Next items to work on
- Any open blockers

#### Audit Gaps (`.claude/audit-gaps.md`)
- First unchecked item for this craft
- Overall progress count

#### Session Log (`.claude/session-log/current.md`)
- Last 3 entries for continuity
- Previous decisions and blockers

#### Debug Logs (`.claude/debug-log/`)
- Any open (non-resolved) investigations for this craft
- Just the title + status, not full investigation details

#### TODOs (`.claude/todos.md`)
- Open items for this craft (grep for craft name in backticks)
- Count by priority level

#### Design Reviews (`.claude/design-reviews/`)
- Most recent review file mentioning this craft
- Just the verdict + any open dispositions (FIX NEEDED / SPEC UPDATE NEEDED)

#### Integration Checks (`.claude/integration-checks/`)
- Most recent check involving this craft
- Just the verdict — PASS or FAIL with specific broken points

### Step 3: Hybrid Search for Patterns

Search `.claude/patterns/` for this craft:
1. Grep for craft name → find relevant pattern files
2. Read the pattern summary (first 10 lines) for semantic relevance
3. Load only patterns that match the current task context

### Step 4: Present Context

Format the extracted context as a concise briefing:

```markdown
## Context: <Craft Name>

### Architecture
<key points from CLAUDE.md/design doc>

### Current Progress
<from feature tracker — what's done, in progress, next>

### Open Issues
<from debug logs — any active investigations>

### Open TODOs
<from todos.md — open items for this craft, count by priority>

### Recent Work
<from session log — last session's summary>

### Relevant Patterns
<from patterns — applicable patterns for this task>
```

## Rules

1. **Never dump full documents** — extract relevant sections only
2. **Prepend source metadata** — always say where context came from
3. **Prefer recent over old** — recent session logs and debug logs first
4. **Skip missing sources gracefully** — if no feature tracker exists yet, note it and continue
5. **Don't block on context** — if sources are missing, load what's available and proceed

## Integration

- **design-check**: context-load feeds design doc sections to design-check
- **feature-track**: context-load reads feature status for session briefing
- **session-log**: context-load reads recent entries for continuity
- **debug-log**: context-load surfaces open investigations
- **todo-track**: context-load reads open TODOs to inform session priorities
- **integration-check**: context-load reads latest check results for this craft
