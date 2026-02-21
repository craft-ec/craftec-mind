# craftec-mind

Craftec's knowledge vault — an Obsidian-native knowledge base used by both Claude Code agents and human developers.

## What is this?

This directory serves double duty:
- **Claude Code's working directory** — skills, debug logs, feature trackers, patterns, and session logs
- **An Obsidian vault** — open this folder in [Obsidian](https://obsidian.md) for visualization, search, and dashboards

## Structure

```
craftec-mind/
├── .obsidian/          Vault configuration (Dataview plugin, theme, etc.)
├── dashboards/         Dataview query pages (features, bugs, integration)
├── templates/          Obsidian templates for manual note creation
├── debug-log/          Investigation logs (structured debugging protocol)
├── features/           Feature-level progress tracking per craft
├── session-log/        Session documentation for cross-session continuity
├── patterns/           Reusable code patterns with contextual metadata
├── skills/             Claude Code skills (hidden in Obsidian sidebar)
└── audit-gaps.md       Current audit progress tracker
```

## Conventions

All generated files follow Obsidian-native markdown:
- **YAML frontmatter** with craft, type, status, created/updated dates, tags
- **Wikilinks** (`[[target]]`) for cross-references between notes
- **Tags** for filtering: `#craftnet`, `#bug`, `#investigating`, etc.

## Dashboards

Open these in Obsidian to see aggregated views:
- `dashboards/features.md` — feature progress across all crafts
- `dashboards/bugs.md` — open investigations and recently resolved bugs
- `dashboards/integration.md` — integration gaps and checks

## Usage

### For Obsidian users
1. Open this folder as a vault in Obsidian
2. Install the Dataview community plugin when prompted
3. Dashboards will auto-populate as content is added

### For Claude Code agents
Skills in `skills/` are automatically available. The vault is populated through skill usage:
- `debug-log` writes to `debug-log/`
- `feature-track` writes to `features/`
- `patterns` writes to `patterns/`
- `session-log` writes to `session-log/`
- `audit` writes to `audit-gaps.md`
