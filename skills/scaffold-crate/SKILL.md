---
name: scaffold-crate
description: Generate a new Rust crate following Craftec conventions — workspace inheritance, thiserror patterns, module structure. Pattern-first approach loads from knowledge base before generating.
user-invocable: true
disable-model-invocation: true
trigger: "create a new crate", "scaffold a daemon", "add a core crate"
layer: work
allowed-tools: Read, Grep, Glob, Bash, Write
---

# scaffold-crate — Crate Generation

## Purpose
Generate new Rust crates that follow Craftec ecosystem conventions exactly. Uses a pattern-first approach — loads from `.claude/patterns/` first, falls back to reading reference files.

## Arguments

```
scaffold-crate <craft-name> <crate-type>
```

- `craft-name`: The craft this crate belongs to (e.g., `craftnet`, `craftobj`)
- `crate-type`: One of: `core`, `daemon`, `client`, `settlement`, `network`, `aggregator`, `cli`

## Protocol

### Step 1: Load Patterns

Check `.claude/patterns/` for:
- `cargo-patterns.md` — Cargo.toml workspace inheritance patterns
- `daemon-patterns.md` — daemon service structure (if crate-type is daemon)
- `error-patterns.md` — thiserror pattern
- `ipc-patterns.md` — IPC dispatch (if crate-type is daemon)

If patterns don't exist, read from reference files instead:

| Crate Type | Reference |
|-----------|-----------|
| core | `craftnet/crates/core/` |
| daemon | `craftnet/crates/daemon/` |
| client | `craftnet/crates/client/` (if exists) |
| network | `craftnet/crates/network/` |
| cli | `craftnet/crates/cli/` (if exists) |
| settlement | `craftec-core/crates/settlement/` |
| aggregator | `craftnet/crates/aggregator/` (if exists) |

### Step 2: Generate Cargo.toml

Use workspace inheritance for common fields:

```toml
[package]
name = "<craft>-<type>"
version.workspace = true
edition.workspace = true
license.workspace = true
repository.workspace = true

[dependencies]
# craftec-core dependencies
craftec-core = { path = "../../craftec-core/crates/core" }  # adjust path
thiserror.workspace = true
tokio = { workspace = true, features = ["full"] }
tracing.workspace = true
serde = { workspace = true, features = ["derive"] }
```

Adjust dependencies based on crate type:
- daemon: add `craftec-ipc`, `craftec-keystore`, `craftec-logging`, `craftec-settings`, `craftec-app`
- network: add `craftec-network`, `libp2p`
- settlement: add `craftec-settlement`
- cli: add `clap`

### Step 3: Generate Source Files

#### `src/lib.rs`
```rust
pub mod error;
// Additional modules based on crate type

pub use error::{Error, Result};
```

#### `src/error.rs`
```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum Error {
    #[error("io error: {0}")]
    Io(#[from] std::io::Error),

    #[error("{0}")]
    Other(String),
}

pub type Result<T> = std::result::Result<T, Error>;
```

#### Additional files by crate type:
- **daemon**: `src/service.rs` (DaemonService with IPC handler), `src/config.rs`
- **cli**: `src/main.rs` (clap arg parser)
- **network**: `src/protocol.rs` (libp2p behaviour)

### Step 4: Update Workspace

Add to root `Cargo.toml`:
1. Add to `[workspace.members]`
2. Add to `[workspace.dependencies]` if the crate should be available to others

### Step 5: Chain

1. Run `dep-check` to verify no dependency violations
2. Run `verify --crate <new-crate>` to confirm it compiles
3. Chain into `commit-guard`

## Rules

1. **Pattern-first** — always check patterns before reading reference files
2. **Workspace inheritance** — never hardcode version, edition, license in crate Cargo.toml
3. **Minimal dependencies** — only add what the crate type needs
4. **craftec-core only** — new crates depend on craftec-core crates, never on sibling crafts
5. **No dead code** — generated stubs should be minimal and compilable

## Integration

- **patterns**: Loads patterns for code generation
- **dep-check**: Validates dependencies after scaffolding
- **verify**: Confirms the new crate compiles
- **commit-guard**: Commits the scaffolded crate
- **new-craft**: Uses scaffold-crate as a worker for each crate in a new craft
