---
type: todos
updated: 2026-02-21T22:00:00Z
open_count: 7
tags: [todos, tracking]
---

# TODOs

## Critical
- [x] ~~**Update ecosystem doc + CLAUDE.md naming**~~ — Fixed: `datacraft`→`craftobj`, `computecraft`→`craftcom`, `parallelcraft`→removed; added all repos to ecosystem doc + CLAUDE.md `ecosystem` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → done 2026-02-21T22:00:00Z
- [x] ~~**Decide aggregator architecture**~~ — DEFERRED: freeze as-is, no changes to CraftNet or CraftOBJ aggregators. Needs bigger design discussion for integration/consolidation. `aggregator` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → deferred 2026-02-21T21:00:00Z
- [x] ~~**Implement craftec-identity key signing ops**~~ — Done: added `pubkey()`, `secret_bytes()`, `sign_bytes()`, `verify_bytes()`, `verify_with_did()`. Social recovery, reputation → deferred (CraftCOM scope) `craftec-core` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → done 2026-02-21T22:00:00Z

## High
- [x] ~~**Namespace data methods in frontend**~~ — Fixed: `publish`→`data.publish`, `fetch`→`data.fetch`, `list`→`data.list`, `status`→`data.status`, `extend`→`data.extend` `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z` → done 2026-02-21T22:00:00Z
- [x] ~~**Add PricingPlan system to settlement SDK**~~ — DEFERRED: skip for now, dedicated settlement session later `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → deferred 2026-02-21T21:00:00Z
- [x] ~~**Add `update_config` instruction to settlement SDK**~~ — DEFERRED: skip for now `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → deferred 2026-02-21T21:00:00Z
- [x] ~~**Add `force_close` instruction to settlement SDK**~~ — DEFERRED: skip for now `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → deferred 2026-02-21T21:00:00Z
- [ ] **Make agent-runtime functional** — WASM sandbox scaffolded but can't run agents `craftec-core` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Write design docs for 5 repos** — craftsec, craftsql, craftvfs, craftcom, scalecraft `ecosystem` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftNet DNS-level blocking at exit** — ExitConfig has blocked_domains but no DNS integration; DNS rebinding protection missing `craftnet` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftStudio identity management UI** — No identity page wired to daemon `craftstudio` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftStudio settlement/wallet UI** — No wallet page wired to daemon `craftstudio` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`

## Medium
- [x] ~~**Update CRAFTSTUDIO_DESIGN.md event format**~~ — Updated: spec now shows standard JSON-RPC notifications (no wrapper) `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z` → done 2026-02-21T22:00:00Z
- [x] ~~**Update CRAFTSTUDIO_DESIGN.md daemon architecture**~~ — Updated: spec now documents in-process architecture with ServerBuilder `craftstudio` [[design-reviews/2026-02-21T183000Z]] `2026-02-21T18:30:00Z` → done 2026-02-21T22:00:00Z
- [x] ~~**Update AGGREGATOR_DESIGN.md**~~ — Updated: added architecture status note — native Rust frozen, CraftCOM is future `aggregator` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → done 2026-02-21T22:00:00Z
- [x] ~~**Reputation computation logic**~~ — DEFERRED: potentially CraftCOM scope `craftec-core` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z` → deferred 2026-02-21T21:00:00Z
- [ ] **CraftNet mobile TUN interface** — uniffi crate exists but incomplete `craftnet` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **Light Protocol double-claim prevention** — Types exist in CraftNet but not in core SDK `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`

## Low
- [ ] **Settlement naming: `fund_creator_pool` vs spec's `deposit`** — Minor naming inconsistency `settlement` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
- [ ] **CraftOBJ segment size not configurable** — Hardcoded to 10 MB `craftobj` [[design-reviews/2026-02-21T120000Z]] `2026-02-21T12:00:00Z`
