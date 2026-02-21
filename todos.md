---
type: todos
updated: 2026-02-21T18:45:00Z
open_count: 3
tags: [todos, tracking]
---

# TODOs

## Critical

## High
- [ ] **Namespace data methods in frontend** — DaemonClient helper methods call `"publish"` not `"data.publish"`, relies on default_handler fallback instead of proper namespacing `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z`

## Medium
- [ ] **Update CRAFTSTUDIO_DESIGN.md event format** — Spec says `{"method":"event","params":{"type":"..."}}` wrapper but impl uses standard JSON-RPC `{"method":"<type>","params":{...}}` which is more correct; update spec to match impl `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z`
- [ ] **Update CRAFTSTUDIO_DESIGN.md daemon architecture** — Spec says separate `craftstudio-daemon` crate; impl uses in-process daemons in Tauri app (deliberate choice for less overhead); update spec to reflect reality `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z`

## Low
