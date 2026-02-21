---
name: integration-check
description: Verify component connections and end-to-end flows. Stops premature "done" declarations by checking that components actually wire together.
user-invocable: true
trigger: Auto when declaring a component or feature "done"; "check integration", "does this actually work end to end"
layer: discipline
allowed-tools: Read, Grep, Glob, Bash, Write, Edit
---

# integration-check — End-to-End Integration Verification

## Purpose
Stop declaring components "done" without verifying they connect to the rest of the system. A component that compiles is NOT a working feature. This skill traces the actual wiring between components.

## Protocol

### Step 1: Identify Connections

Read the component's code and determine:
- **What calls this component** (upstream)
- **What this component calls** (downstream)
- **What data flows through it** (types, structs, enums)

Use Grep to find:
- `use <crate>::` — who imports this component
- Function calls to this component's public API
- Trait implementations that connect components

### Step 2: Check Upstream (What Calls This)

For each caller:
- Verify the call site exists (match arm, function call, dispatch entry)
- Verify the arguments match the function signature
- Verify error handling is present (not just `.unwrap()`)
- Flag missing call sites

Report format:
```markdown
## Integration Check: <component>

### Upstream (what calls this)
- <file>: <caller> calls <method>
  → ✅ Call exists, signatures match
  → ❌ Missing: <expected caller> does not invoke this
  → ⚠️ Signature mismatch: caller passes X, component expects Y
```

### Step 3: Check Downstream (What This Calls)

For each dependency:
- Verify the called function/method exists
- Verify the types match (structs passed between components are the same, not duplicated)
- Verify the component actually makes the call (not just importing the dependency)

```markdown
### Downstream (what this calls)
- <dependency>: <function>
  → ✅ Function exists and signatures match
  → ❌ Function exists but component never calls it
  → ⚠️ Component calls it but return type is ignored
```

### Step 4: Verify Data Flow

Trace the complete data path for the feature:
```markdown
### Data Flow
- <start> → <step 1> → <step 2> → ... → <end>
  → ✅ Complete chain verified
  → ❌ BROKEN at <step X> → <step Y>: <why>
```

### Step 5: Check for Dead Code

Flag public functions/methods that nothing calls:
- Either the integration is missing (caller needs to be added)
- Or the function shouldn't be public (make it private or remove it)

### Step 6: Verify IPC Wiring (for daemon components)

For each IPC method defined in the design doc, check:
1. Match arm exists in the handler dispatch
2. Service method exists and is called by the match arm
3. Request/response types are defined
4. If the method needs a Command enum variant, it exists

### Step 7: Report and Block

Generate a clear pass/fail report:
- **PASS**: All connections verified, data flows complete
- **FAIL**: List all gaps with specific file paths and line numbers

If the check FAILs:
- Block "done" status in `feature-track`
- The component is "built" but not "integrated"

### Step 8: End-to-End Flow (for features)

For feature-level checks, trace the COMPLETE user flow:
1. User action (CLI command, IPC call, UI interaction)
2. Through each processing layer
3. To the final result (response, side effect, settlement)

Identify any gaps in the chain.

### Step 9: Record the Check (MANDATORY)

**Every integration check MUST be recorded** to `.claude/integration-checks/` as a per-session file.

**File naming**: `.claude/integration-checks/<YYYY-MM-DDTHHMMSS>Z.md`

**Format**:

```markdown
---
type: integration-check
timestamp: 2026-02-21T18:30:00Z
components: [<component-1>, <component-2>]
verdict: PASS | FAIL
tags: [integration-check, <craft>, <component>]
---

# Integration Check: <YYYY-MM-DDTHH:MM:SSZ>

**Context**: <what triggered this check>

## <component>
<the full report from Steps 2-8>
```

### Step 10: Extract TODOs

After recording, extract all **FAIL** items and broken data flows to `.claude/todos.md` using the `todo-track` format with source link `[[integration-checks/<filename>]]`.

## Integration

- **feature-track**: Blocks Done status if integration check fails
- **audit**: Includes integration checks in audit scope
- **verify**: Compilation success ≠ integration success; this skill checks what verify cannot
- **todo-track**: Receives failed integration points as action items
