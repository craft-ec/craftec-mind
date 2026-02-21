---
name: debug-log
description: Structured investigation protocol with file-based log trail. Stops blind guessing — every debugging attempt is logged with hypothesis, test, and result.
user-invocable: true
trigger: Auto when any error/failure is encountered; "debug this", "investigate", "why is this failing"
layer: discipline
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---

# debug-log — Structured Investigation Protocol

## Purpose
Stop agents from guessing at fixes. Every debugging attempt MUST be logged with hypothesis → evidence → test → result. No fix without confirmed root cause.

## Protocol

When an error or test failure is encountered, BEFORE attempting any fix:

### Step 1: Log the Error

Create a file at `.claude/debug-log/<descriptive-name>.md` using this format:

```markdown
---
craft: <craft-name>
component: <component-name>
type: debug-log
status: investigating
created: <ISO 8601 timestamp>
updated: <ISO 8601 timestamp>
tags: [bug, <craft>, <component>]
related: []
---
# <Issue Title>

**Error**: <exact error message, copied verbatim>
**Context**: <what was being done when the error occurred>
**Design ref**: <link to relevant design doc section if applicable>

## Investigation

### Attempt 1
**Hypothesis**: <what you think the cause is and WHY>
**Evidence**: <what code/logs/behavior supports this hypothesis>
**Test**: <specific test to confirm or reject the hypothesis>
**Result**: <CONFIRMED or REJECTED — what actually happened>
**Files examined**: <list of files read during investigation>

## Root Cause
<identified ONLY after evidence confirms it>

## Fix
<the fix, with rationale tied to root cause>
```

### Step 2: Read Before Guessing

Before forming ANY hypothesis:
1. Read the relevant source code — especially error handling paths
2. Read the design doc for the component (check `docs/` directory)
3. Search `.claude/debug-log/` for previous investigations of similar issues
4. Check `.claude/patterns/` for known patterns that apply

### Step 3: Test the Hypothesis

Do NOT apply a fix just to see if it works. Instead:
1. Write a minimal reproduction or targeted test
2. Or trace the code path manually and identify the exact failure point
3. Log the test and its result in the debug log

### Step 4: Max 3 Hypotheses

If 3 hypotheses have been tested and rejected:
- **STOP** — do not continue guessing
- Update the debug log with all findings so far
- Ask the user for guidance, providing the debug log path
- The 3-hypothesis limit prevents infinite loops of guessing

### Step 5: Fix Only After Confirmed Root Cause

The fix MUST:
- Directly address the confirmed root cause
- NOT paper over symptoms (no blind try/catch, no ignoring errors)
- Be documented in the debug log with rationale

### Step 6: Update Status

After fixing:
- Update the debug log `status` field to `resolved`
- Update the `updated` timestamp
- Add the fix details to the Fix section

### Step 7: Extract TODOs

If the root cause reveals issues that can't be fixed immediately, add them to `.claude/todos.md` using the `todo-track` format with source link `[[debug-log/<investigation-name>]]`.

## Cross-Session Value

Debug logs persist on disk. When a similar error occurs in a future session:
1. `context-load` or grep finds the previous investigation
2. The previous root cause and fix are immediately available
3. No repeated work — the investigation is done once

## Examples

**Good investigation log name**: `craftnet-relay-signature-panic.md`, `ipc-timeout-on-large-payload.md`
**Bad name**: `bug1.md`, `error.md`, `fix.md`

## Integration

- **verify** skill: Falls back to debug-log protocol after 2 failed auto-fix retries
- **audit** skill: Uses debug-log for persistent failures during batch fixes
- **feature-track**: Debug logs can be linked from feature blockers
- **todo-track**: Receives deferred fixes from investigations that identify issues beyond current scope
