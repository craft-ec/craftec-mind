---
name: session-log
description: Session documentation and archival for cross-session continuity. Logs what was done, decisions made, blockers hit, and what should happen next.
user-invocable: true
trigger: "log this", "save session"; auto on handover or end of significant work
layer: knowledge
allowed-tools: Read, Write
---

# session-log — Session Documentation

## Purpose
Maintain a persistent log of work sessions for cross-session continuity. When a new session starts, `context-load` reads the recent entries to understand what was done, what decisions were made, and what should happen next.

## Data Store

`.claude/session-log/current.md` — single file with entries prepended (newest first)

## Protocol

### Writing an Entry

When the user says "log this", "save session", or when a significant unit of work is completed:

1. Read `.claude/session-log/current.md`
2. Prepend a new entry below the header comment:

```markdown
---
type: session
craft: <craft-name or "multi">
created: <today's date>
tags: [session, <craft>]
---

## <date> <time> — <One-line Summary>
**What**: <concise description of what was done>
**Changed**:
- <file/crate 1>: <what changed>
- <file/crate 2>: <what changed>

**Decisions**:
- <key decision and why it was made>

**Blockers**:
- <any blockers hit, with links to debug logs if applicable>

**Next**:
- <what should happen next, in priority order>

**Related**: [[features/<craft>#<feature>]], [[debug-log/<investigation>]]

---
```

### Archival

When `current.md` exceeds 100 entries (approximately):
- Move older entries to `.claude/session-log/archive-<year>-<month>.md`
- Keep the 20 most recent entries in `current.md`

### Handover Notes

When explicitly ending a work session or handing over to another agent:
1. Write a session-log entry summarizing the entire session
2. Include a clear "Next" section with prioritized action items
3. Reference all relevant feature trackers and debug logs
4. Chain into `commit-guard` to persist the log

## Rules

1. **Prepend, don't append** — newest entries first for quick reading
2. **Be concise but complete** — enough detail to reconstruct context in a new session
3. **Always include Next** — the most important field; tells the next session where to start
4. **Link related artifacts** — wikilinks to features, debug logs, patterns
5. **One entry per logical unit** — not one per file change; one per meaningful work block

## Integration

- **context-load**: Reads the last 3 entries for session continuity
- **commit-guard**: session-log chains into commit-guard to persist entries
- **feature-track**: Session entries reference feature progress
- **debug-log**: Session entries link to active investigations
