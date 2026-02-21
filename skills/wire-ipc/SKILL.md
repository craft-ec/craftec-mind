---
name: wire-ipc
description: Add an IPC method to a craft daemon — generates request/response types, handler match arm, and service method. Checks design doc for method specification first.
user-invocable: true
disable-model-invocation: true
trigger: "add an IPC method", "wire up get_status", "add a new RPC method"
layer: work
allowed-tools: Read, Grep, Glob, Write, Edit
---

# wire-ipc — IPC Method Wiring

## Purpose
Add a complete IPC method to a craft's daemon, following the established pattern. Generates all the pieces needed for a working JSON-RPC method: types, handler dispatch, and service implementation.

## Arguments

```
wire-ipc <craft-name> <method-name> [description]
```

- `craft-name`: Which craft's daemon to wire (e.g., `craftnet`, `craftobj`)
- `method-name`: The RPC method name (e.g., `get_status`, `relay.start`)
- `description`: Optional description of what the method does

## Protocol

### Step 1: Design Check

Before generating anything, check the design doc:
1. Read `docs/<CRAFT>_DESIGN.md`
2. Search for the method name or its functionality
3. If specified in the design doc:
   - Use the spec's request/response types
   - Use the spec's method name exactly
   - Note any validation or security requirements
4. If NOT in the design doc:
   - Flag to the user: "Method `<name>` is not in the design doc. Proceed anyway?"
   - If proceeding, note it as an addition to be documented

### Step 2: Detect IPC Pattern

Read the existing daemon code to understand the pattern used:
- Find the handler dispatch file (usually `src/service.rs` or `src/handler.rs`)
- Identify the match pattern for method routing
- Identify where request/response types are defined
- Check for a Command enum pattern

Common patterns:
- **Direct match**: `match method { "get_status" => self.get_status(params), ... }`
- **Command enum**: `Command::GetStatus(params) => ...` with separate dispatch
- **Namespace prefix**: `"tunnel.get_status"` with namespace stripping

### Step 3: Generate Request/Response Types

In the appropriate types file (usually `src/types.rs` or alongside the handler):

```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct <MethodName>Request {
    // fields from design doc or user description
}

#[derive(Debug, Serialize, Deserialize)]
pub struct <MethodName>Response {
    // fields from design doc or user description
}
```

### Step 4: Add Handler Match Arm

In the dispatch function, add a match arm following the existing pattern:

```rust
"<method.name>" => {
    let params: <MethodName>Request = serde_json::from_value(params)?;
    let result = self.<method_name>(params).await?;
    Ok(serde_json::to_value(result)?)
}
```

### Step 5: Add Service Method

In the service implementation:

```rust
async fn <method_name>(&self, req: <MethodName>Request) -> Result<<MethodName>Response> {
    // TODO: implement
    todo!("<method_name> not yet implemented")
}
```

### Step 6: Chain

1. Run `verify --crate <craft>-daemon` to confirm compilation
2. Chain into `commit-guard`

## Rules

1. **Design-check first** — always check the spec before generating
2. **Follow existing patterns** — match the craft's existing IPC style exactly
3. **No empty implementations** — use `todo!()` for stubs, not empty functions that silently succeed
4. **Namespace correctly** — if the craft uses namespaced methods (`tunnel.*`), follow the convention
5. **Add both transports** — if the daemon supports both socket and WebSocket, wire for both

## Integration

- **design-check**: Validates method against spec before generating
- **verify**: Confirms the daemon still compiles after wiring
- **commit-guard**: Commits the new method
- **patterns**: Loads IPC patterns for reference
- **feature-track**: Updates the feature tracker with the new integration point
