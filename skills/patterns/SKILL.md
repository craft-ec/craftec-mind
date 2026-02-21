---
name: patterns
description: Save and search reusable code patterns with contextual metadata. The living template library — patterns are enriched with source, problem context, and craft applicability.
user-invocable: true
disable-model-invocation: true
trigger: "save pattern", "remember this pattern", "load pattern", "what's the pattern for..."
layer: knowledge
allowed-tools: Read, Grep, Glob, Write
---

# patterns — Context-Enriched Pattern Library

## Purpose
Maintain a living library of reusable code patterns extracted from the Craftec ecosystem. Patterns are enriched with metadata (source file, problem context, applicable crate types) so they can be found by hybrid search and applied correctly.

## Data Store

Files in `.claude/patterns/` organized by category:
- `ipc-patterns.md` — IPC dispatch, handler, request/response patterns
- `error-patterns.md` — thiserror definitions, error conversion, error handling
- `daemon-patterns.md` — daemon service setup, command dispatch, shutdown
- `test-patterns.md` — test setup, fixtures, integration test patterns
- `cargo-patterns.md` — Cargo.toml workspace inheritance, feature flags, dependencies

## Protocol

### Saving a Pattern

When the user says "save pattern", "remember this pattern", or when a reusable pattern is identified:

1. **Identify the category** from the pattern content
2. **Extract the pattern** with context
3. **Append to the appropriate file** (or create it if it doesn't exist)

Format for each pattern entry:

```markdown
---
type: pattern
category: <category>
craft: <craft-name or "all">
source: <file path where this was extracted from>
tags: [pattern, <category>, <craft>]
created: <today's date>
---
## <Pattern Name>
**Applies to**: <crate types where this pattern is used>
**Problem**: <what problem this pattern solves>
**See also**: [[patterns/<related>]], [[features/<craft>#<feature>]]

### Pattern
\```rust
<the code pattern with placeholders where values vary>
\```

### Usage Notes
<when to use this, gotchas, variations>

### Example
<concrete usage from an actual crate>
```

### Loading a Pattern

When the user asks "what's the pattern for..." or "load pattern":

1. **Keyword search**: Grep `.claude/patterns/` for the keywords
2. **Read summaries**: For each match, read the pattern name + "Applies to" + "Problem" lines
3. **Semantic filter**: Present only patterns relevant to the current context
4. **Return the pattern**: Show the code pattern with any needed adaptations noted

### Hybrid Search

Combine two approaches:
1. **Grep**: Search for exact keywords in pattern files (fast, precise)
2. **Semantic**: Read pattern summaries and match against the task context (catches patterns that don't share keywords)

## Rules

1. **One pattern per concern** — don't bundle unrelated patterns in one entry
2. **Always include source** — where was this pattern observed? (file path)
3. **Always include problem** — what does this pattern solve?
4. **Use placeholders** — `<craft_name>`, `<method_name>`, not hardcoded values
5. **Update, don't duplicate** — if a pattern already exists, update it; don't create a second entry
6. **Patterns are living** — update them when the ecosystem evolves

## Integration

- **scaffold-crate**: Loads patterns before generating code (pattern-first approach)
- **wire-ipc**: Loads IPC patterns for method generation
- **context-load**: Searches patterns for craft-relevant entries
- **commit-guard**: Chains into commit-guard after saving patterns
