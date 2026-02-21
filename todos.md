---
type: todos
updated: 2026-02-21T20:30:00Z
open_count: 18
tags: [todos, tracking]
---

# TODOs

## Critical
- [ ] **Update ecosystem doc + CLAUDE.md naming** — `datacraft`→`craftobj`, `computecraft`→`craftcom`, `parallelcraft`→removed; add all 16+ repos to ecosystem doc `ecosystem` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Decide aggregator architecture** — Design says CraftCOM WASM agent + CraftSQL; impl is native Rust + P2P gossipsub. Both CraftNet and CraftOBJ aggregators diverge. Decide and update design doc. `aggregator` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Implement craftec-identity key features** — Social recovery (M-of-N guardians), key rotation (preserves DID), cross-craft reputation aggregation, identity profiles `craftec-core` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`

## High
- [ ] **Namespace data methods in frontend** — DaemonClient helper methods call `"publish"` not `"data.publish"`, relies on default_handler fallback instead of proper namespacing `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z`
- [ ] **Add PricingPlan system to settlement SDK** — Admin-defined tiers with billing periods; `create_plan`, `update_plan`, `delete_plan` instructions missing `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Add `update_config` instruction to settlement SDK** — Admin config updates missing from instruction builders `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Add `force_close` instruction to settlement SDK** — Unilateral payment channel close missing; only cooperative close exists `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Make agent-runtime functional** — WASM sandbox scaffolded but can't run agents; prerequisite for CraftCOM aggregator architecture `craftec-core` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Write design docs for 5 repos** — craftsec, craftsql, craftvfs, craftcom, scalecraft all need design documents `ecosystem` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftNet DNS-level blocking at exit** — ExitConfig has blocked_domains but no DNS integration; DNS rebinding protection missing `craftnet` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftStudio identity management UI** — No identity page wired to daemon `craftstudio` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftStudio settlement/wallet UI** — No wallet page wired to daemon `craftstudio` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`

## Medium
- [ ] **Update CRAFTSTUDIO_DESIGN.md event format** — Spec says `{"method":"event","params":{"type":"..."}}` wrapper but impl uses standard JSON-RPC `{"method":"<type>","params":{...}}`; update spec to match impl `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z`
- [ ] **Update CRAFTSTUDIO_DESIGN.md daemon architecture** — Spec says separate `craftstudio-daemon` crate; impl uses in-process daemons in Tauri app; update spec to reflect reality `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z`
- [ ] **Reputation computation logic** — ReputationScore struct exists but no actual computation; needs cross-craft aggregation `craftec-core` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftNet mobile TUN interface** — uniffi crate exists but incomplete; iOS Network Extension not wired `craftnet` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Light Protocol double-claim prevention** — Types exist in CraftNet settlement but not in core SDK; move to shared crate `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`

## Low
- [ ] **Settlement naming: `fund_creator_pool` vs spec's `deposit`** — Minor naming inconsistency `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftOBJ segment size not configurable** — Hardcoded to 10 MB; spec says configurable `craftobj` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
