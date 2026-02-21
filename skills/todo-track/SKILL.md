---
name: todo-track
description: Centralized TODO tracking. Captures action items from design reviews, audits, debug sessions, and implementation work. Single file, never lost.
user-invocable: true
trigger: "add todo", "show todos", "what needs doing", "open items"; auto after design-check, audit, debug-log
layer: discipline
allowed-tools: Read, Write, Edit, Grep, Glob
---

# todo-track — Centralized TODO Tracking

## Purpose
Action items generated during design reviews, audits, debugging, and implementation get buried in session documents and lost across context compactions. This skill ensures every TODO is captured in one queryable file that survives sessions.

## Data Store

**Single file**: `.claude/todos.md`

One file because:
- TODOs are actively worked down — the file should shrink, not grow forever
- You need to see ALL open items across the project at a glance
- Grep for open items is trivial (`- [ ]`)
- Completed items provide history of what was resolved

## Protocol

### Adding TODOs

When action items are identified (from any source), add them to `.claude/todos.md`.

**Sources that MUST generate TODOs**:
- `design-check` dispositions marked **FIX NEEDED** or **SPEC UPDATE NEEDED**
- `audit` gap items that need fixing
- `debug-log` investigations that identify root causes needing fixes
- `integration-check` failures that need resolution
- User-requested tasks that can't be done immediately

**Format**: Each TODO is a checkbox item in the appropriate priority section:

```markdown
---
type: todos
updated: 2026-02-21T18:30:00Z
open_count: 5
tags: [todos, tracking]
---

# TODOs

## Critical
- [ ] **<title>** — <description> `<component>` [[<source>]] `<timestamp>`
- [x] ~~**<title>** — <description>~~ `<component>` [[<source>]] `<timestamp>` → done <timestamp>

## High
- [ ] **<title>** — <description> `<component>` [[<source>]] `<timestamp>`

## Medium
- [ ] **<title>** — <description> `<component>` [[<source>]] `<timestamp>`

## Low
- [ ] **<title>** — <description> `<component>` [[<source>]] `<timestamp>`
```

**Field descriptions**:

| Field | Description |
|-------|-------------|
| `title` | Short imperative description (e.g., "Namespace data methods in frontend") |
| `description` | One-line context of what needs to change |
| `component` | Affected component in backticks (e.g., `craftstudio`, `craftec-ipc`) |
| `source` | Obsidian wikilink to the originating document (e.g., `[[design-reviews/2026-02-21T183000Z]]`) |
| `timestamp` | ISO 8601 when the TODO was created |

### Priority levels

| Priority | When to use |
|----------|-------------|
| **Critical** | Blocks other work, security issue, broken functionality |
| **High** | FIX NEEDED from design-check, failing integration points |
| **Medium** | SPEC UPDATE NEEDED, cleanup, naming consistency |
| **Low** | Nice-to-have improvements, cosmetic issues |

### Completing TODOs

When a TODO is resolved:
1. Change `- [ ]` to `- [x]`
2. Strikethrough the title: `~~**title**~~`
3. Append ` → done <timestamp>` at the end
4. Update the frontmatter `updated` timestamp and `open_count`

Do NOT delete completed items immediately — they provide history. Periodically (monthly or when the file exceeds 100 items), move completed items to `.claude/todos-archive.md`.

### Reviewing TODOs

When invoked with "show todos" or "what needs doing":
1. Read `.claude/todos.md`
2. Report open count by priority
3. Highlight any that are stale (created > 7 days ago, still open)
4. Suggest which to tackle based on current session context

### Creating the file

If `.claude/todos.md` doesn't exist, create it:

```markdown
---
type: todos
updated: <current ISO 8601 timestamp>
open_count: 0
tags: [todos, tracking]
---

# TODOs

## Critical

## High

## Medium

## Low
```

## Integration

- **design-check**: Step 7 extracts FIX NEEDED / SPEC UPDATE NEEDED → TODO items
- **audit**: Gap items become TODOs
- **debug-log**: Root causes identified during debugging become TODOs
- **integration-check**: Failed integration points become TODOs
- **feature-track**: Blocked features reference TODOs
- **context-load**: Reads TODOs to inform session priorities
