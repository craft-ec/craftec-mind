---
type: audit
status: complete
created: 2026-02-21T12:00:00Z
updated: 2026-02-21T18:30:00Z
tags: [audit, design-review, all-crafts]
---
# Design vs Implementation Gap Analysis

**Timestamp**: 2026-02-21T12:00:00Z
**Scope**: All design docs in `docs/` vs actual implementation across all repos

---

## Executive Summary

The Craftec ecosystem has **9 design documents** and **16+ repositories**. The core infrastructure (`craftec-core`, `craftnet`, `craftobj`) is substantially implemented. However, significant gaps exist between what the design docs specify and what's actually built — particularly in settlement, CraftStudio, and the newer craft layers (CraftSQL, CraftCOM).

**Key finding**: Several repos exist that have NO corresponding design doc, and the ecosystem doc is outdated (still references `computecraft`/`parallelcraft` but actual repos are `craftcom`/`craftgpu`).

---

## 1. CRAFTEC ECOSYSTEM (`docs/CRAFTEC_ECOSYSTEM.md`)

### Spec Says vs Reality

| Spec Claims | Reality | Status |
|-------------|---------|--------|
| Projects: CraftNet, CraftOBJ, ComputeCraft, ParallelCraft, CraftStudio | Actual repos: craftnet, craftobj, craftcom, craftsec, craftsql, craftvfs, cloakcraft, craftgpu, scalecraft, worldcraft, handcraft, flowcraft, flowcraft-sdk, craftstudio | ❌ OUTDATED |
| `computecraft/` directory | Does not exist; replaced by `craftcom/` | ❌ DEVIATION |
| `parallelcraft/` directory | Does not exist; replaced by `craftgpu/` (gitignored/removed) | ❌ DEVIATION |
| `datacraft/` in CLAUDE.md | Does not exist; actual repo is `craftobj/` | ❌ DEVIATION |
| craftec-core has 13 crates | Actual: 16 crates (adds `aggregator`, `agent-runtime`, `distribution-guest`) | ⚠️ OUTDATED |
| Identity: social recovery, M-of-N guardians, key rotation | `craftec-identity`: DID + reputation score stubs only. NO social recovery, NO guardian system, NO key rotation | ❌ MISSING |
| IPC: WebSocket transport for UI | `craftec-ipc`: Unix socket only. NO WebSocket server | ❌ MISSING |

### Missing from Ecosystem Doc
- **CraftSQL** — distributed relational database (exists, no design doc)
- **CraftVFS** — POSIX distributed filesystem (exists, no design doc)
- **CraftSEC** — threshold signing & attestation (exists, no design doc)
- **CraftCOM** — WASM agent platform (exists, partially described in AGGREGATOR_DESIGN)
- **CloakCraft** — privacy DeFi (deployed Solana program, no design doc)
- **FlowCraft** — subscription streaming (deployed, no design doc)
- **HandCraft** — creator platform (deployed, no design doc)
- **ScaleCraft** — reputation & disputes (exists, no design doc)
- **WorldCraft** — game (exists, no design doc)

---

## 2. CRAFTNET (`docs/CRAFTNET_DESIGN.md`)

### ✅ Well-Implemented (Matches Spec)
- Onion routing with X25519-ECDH + ChaCha20-Poly1305 per-layer
- Erasure coding (Reed-Solomon 5/3, 3KB chunks)
- ForwardReceipt with ed25519 signatures
- Trustless relay verification via encrypted routing tags
- SOCKS5 proxy (TCP tunnel mode)
- HopMode: Direct/Single/Double/Triple/Quad
- Subscription tier enforcement
- Multi-hop path building with RequestBuilder
- Exit node HTTP + TCP tunnel handlers
- Aggregator with append-only history ledger
- Out-of-order proof buffering (blockchain orphan-style)
- Bandwidth time-series indexing + compaction
- Comprehensive test suite (unit + integration + e2e)

### ⚠️ Deviations
| Spec | Implementation | Severity |
|------|----------------|----------|
| Aggregator should be CraftCOM WASM agent (per AGGREGATOR_DESIGN) | Aggregator is a native Rust crate in `craftnet/crates/aggregator/` | ⚠️ MEDIUM — design doc says CraftCOM agent, impl is standalone |
| IPC should support WebSocket for CraftStudio | Daemon only has Unix socket IPC | ⚠️ MEDIUM |

### ❌ Missing
| Feature | Status | Notes |
|---------|--------|-------|
| DNS-level blocking at exit | Not implemented | ExitConfig has blocked_domains but no DNS integration |
| DNS rebinding protection | Not implemented | Only basic IP range SSRF checks |
| Mobile TUN interface | uniffi crate exists but incomplete | iOS Network Extension not wired |

---

## 3. CRAFTOBJ (`docs/CRAFTOBJ_DESIGN.md`)

### ✅ Well-Implemented
- Content-addressed storage with ContentId (SHA-256)
- RLNC erasure coding over GF(2^8) with coefficient vectors
- Self-describing pieces (PieceHeader carries all metadata)
- PDP (Proof of Data Possession) via GF(2^8) cross-checking
- Health scanning + rotational repair
- Demand-driven scaling (hot content replication)
- Persistent streams (two-unidirectional, Bitswap-style)
- Peer exchange protocol (PEX)
- Capability-based peer selection
- Storage receipts for settlement
- DHT-based content routing
- 24K lines across 6 crates, 166+ tests passing

### ⚠️ Deviations
| Spec | Implementation | Severity |
|------|----------------|----------|
| Aggregator as CraftCOM WASM agent reading from CraftSQL | Aggregator is `craftec-core/crates/aggregator/` (native library) | ⚠️ MEDIUM |
| Receipts delivered via CraftSQL | Receipts delivered via P2P gossipsub | ⚠️ MEDIUM |
| `SEGMENT_SIZE` should be configurable | Hardcoded to 10 MB | 🔵 LOW |

### ❌ Missing (Intentionally Removed per Daemon CLAUDE.md)
These were moved to COM layer and are NOT implementation gaps:
- TransferReceipt (no per-byte egress pricing)
- Payment channels (subscriptions replace them)
- Access control + PRE (proxy re-encryption)
- Content removal notices (natural degradation via unpin)
- Creator signing (COM layer)
- Per-byte egress pricing

---

## 4. SETTLEMENT (`docs/SETTLEMENT_DESIGN.md` + `docs/SETTLEMENT_PROGRAM_DESIGN.md`)

### ✅ Implemented in `craftec-core/crates/settlement/`
- ServiceType enum (Tunnel=0, Data=1, Compute=2, Parallel=3, Bundle=4)
- CreatorPool, SubscriptionPool, PaymentChannel account types
- PDA derivation (config, treasury, pool, creator_pool, payment_channel, claim_receipt)
- Instruction builders (13 types)
- StorageTier enum (Free, Lite, Standard, Pro, Enterprise)
- StorageReceipt implementing ContributionReceipt trait

### ⚠️ Deviations
| Spec | Implementation | Severity |
|------|----------------|----------|
| `update_config` instruction | Not found in instruction builders | ⚠️ MEDIUM |
| `deposit` instruction (fan funding for creator pools) | `fund_creator_pool` exists but different name | 🔵 LOW (naming) |
| `force_close` (payment channel timeout close) | Only `close_payment_channel` exists | ⚠️ MEDIUM |
| PricingPlan account with billing periods | No PricingPlan in SDK types | ❌ MISSING |
| `create_plan`, `update_plan`, `delete_plan` instructions | Not in SDK instruction builders | ❌ MISSING |

### ❌ Missing
| Feature | Status | Notes |
|---------|--------|-------|
| PricingPlan system (admin-defined tiers with billing periods) | Not implemented in SDK | Design doc specifies this as admin-managed |
| `update_config` instruction | Missing | Admin config updates |
| `force_close` (unilateral payment channel close) | Missing | Only cooperative close exists |
| Light Protocol integration for double-claim prevention | Types exist in CraftNet settlement but not in core SDK | Partial |

### On-Chain Program (`craftec-settlement/`)
- Separate Solana Anchor program repo exists
- Need to verify instruction set matches SDK builders

---

## 5. PROVER (`docs/PROVER_DESIGN.md`)

### ✅ Well-Implemented
- Dual-mode: Mock (default) and SP1 (feature flag)
- Mock mode: local execution, magic bytes, no real proof
- SP1 mode: real Groth16 proof generation via SP1 SDK
- Guest program: sort entries → compute weights → SHA-256 Merkle tree → output root
- DistributionInput/DistributionOutput types (76 bytes)
- Host-side receipt pre-aggregation (validate, dedup, sum per operator)
- BatchProof structure (proof_bytes + public_inputs)

### ⚠️ Minor Notes
| Spec | Implementation | Severity |
|------|----------------|----------|
| Guest should verify ed25519 signatures | Guest intentionally skips verification (design decision, documented) | ✅ INTENTIONAL |
| `distribution-guest` SP1 binary | Exists but may need rebuild for current SP1 version | 🔵 LOW |

### Status: **~95% complete**

---

## 6. AGGREGATOR (`docs/AGGREGATOR_DESIGN.md`)

### ⚠️ Major Architectural Deviation

The design doc explicitly states:
> "Architecture decision (finalized): The aggregator is a **CraftCOM agent** (`craftobj-aggregator`), not a standalone P2P service. Receipts are delivered via **CraftSQL**, not via a P2P `ReceiptPush` wire message."

**Reality**: `craftec-core/crates/aggregator/` is a native Rust library with:
- ReceiptCollector (P2P gossipsub, NOT CraftSQL)
- BatchBuilder (in-memory, NOT SQL queries)
- DistributionPoster (direct Solana RPC, matches spec)

**Also**: CraftNet has its own aggregator in `craftnet/crates/aggregator/` with:
- Append-only history ledger
- Bandwidth time-series indexing
- Out-of-order proof buffering
- This is NOT a CraftCOM agent either

### Gap Summary
| Spec | Reality | Severity |
|------|---------|----------|
| CraftCOM WASM agent | Native Rust library | ❌ MAJOR |
| Receipts from CraftSQL | Receipts from P2P gossipsub | ❌ MAJOR |
| `sql_execute("INSERT INTO craftobj_receipts")` | In-memory receipt buffer | ❌ MAJOR |
| Crash recovery via SQL persistence | JSON state file (craftnet) / in-memory (core) | ⚠️ MEDIUM |

**Question for user**: Is the design doc ahead of implementation (aspirational), or has the architecture shifted since the doc was written? The current P2P approach works; the CraftCOM/CraftSQL approach is more elegant but requires those systems to be production-ready.

---

## 7. CRAFTSTUDIO (`docs/CRAFTSTUDIO_DESIGN.md`)

### ✅ Exists
- Tauri v2 project structure
- React frontend scaffolded

### ❌ Largely Unimplemented
| Spec Feature | Status |
|-------------|--------|
| `craftstudio-daemon` with libp2p + multi-protocol | Not found / minimal |
| WebSocket JSON-RPC server | Not found |
| Tauri Commands (keychain, system proxy, file dialogs) | Scaffolded but minimal |
| CraftNet integration (tunnel.connect, etc.) | Not wired |
| CraftOBJ integration (data.publish, etc.) | Not wired |
| Identity management UI | Not implemented |
| Settlement/wallet UI | Not implemented |
| System tray | Scaffolded |

### Status: **~15% complete** — foundational Tauri + React scaffolding exists, but no protocol integration

---

## 8. CRAFTEC-CORE INTERNAL GAPS

### `craftec-identity`
| Spec | Status | Notes |
|------|--------|-------|
| DID: `did:craftec:<base58(ed25519_pubkey)>` | ✅ Implemented | |
| DID Document derivation | ✅ Implemented | |
| Reputation score | ⚠️ Stub only | Fields exist, no computation logic |
| Social recovery (M-of-N guardians) | ❌ NOT IMPLEMENTED | Design doc specifies this |
| Key rotation (preserves DID) | ❌ NOT IMPLEMENTED | |
| Cross-craft reputation aggregation | ❌ NOT IMPLEMENTED | |
| Identity profiles (display name, avatar) | ❌ NOT IMPLEMENTED | |

### `craftec-ipc`
| Spec | Status | Notes |
|------|--------|-------|
| JSON-RPC 2.0 server (Unix socket) | ✅ Implemented | |
| JSON-RPC 2.0 client (Unix socket) | ✅ Implemented | |
| WebSocket server (for CraftStudio UI) | ❌ NOT IMPLEMENTED | Critical for CraftStudio |
| Server-sent notifications (events) | ❌ NOT IMPLEMENTED | |
| Named Pipe (Windows) | ⚠️ Path generation exists, server doesn't use it | |

### `craftec-agent-runtime`
| Feature | Status |
|---------|--------|
| WASM sandbox | ~50% — structure exists |
| Permission tiers | Defined (TIER1, TIER2) |
| Governance config | Struct exists |
| Agent whitelist | Struct exists |
| Agent lifecycle (manifest, registry) | Scaffolded |
| Host functions (store, memory) | Scaffolded |
| **Actually running WASM agents** | ❌ NOT FUNCTIONAL |

### `craftec-network`
| Feature | Status |
|---------|--------|
| Kademlia DHT (primary) | ✅ |
| Kademlia DHT (secondary/dual) | ✅ |
| Gossipsub | ✅ |
| Rendezvous (client + server) | ✅ |
| Relay + DCUtR + AutoNAT | ✅ |
| mDNS local discovery | ✅ |
| libp2p-stream | ✅ |
| PEX (peer exchange) | ✅ |

---

## 9. REPOS WITH NO DESIGN DOC

These repos exist in the workspace but have NO corresponding design document in `docs/`:

| Repo | What It Is | Needs Design Doc? |
|------|-----------|-------------------|
| `craftsec/` | Threshold signing & attestation (3 phases complete) | ✅ YES |
| `craftsql/` | Distributed relational database with SQLite VFS | ✅ YES |
| `craftvfs/` | POSIX-like distributed filesystem | ✅ YES |
| `craftcom/` | WASM agent platform | ✅ YES (referenced in AGGREGATOR_DESIGN but no own doc) |
| `cloakcraft/` | Privacy DeFi (deployed Solana program) | ⚠️ MAYBE (deployed, may be stable) |
| `flowcraft/` | Subscription streaming (deployed) | ⚠️ MAYBE |
| `handcraft/` | Creator platform (deployed) | ⚠️ MAYBE |
| `scalecraft/` | Reputation & dispute resolution | ✅ YES |
| `worldcraft/` | Open-world martial arts RPG | 🔵 NO (game, not infra) |

---

## 10. ROOT CLAUDE.md vs REALITY

The root `CLAUDE.md` (project instructions) has its own discrepancies:

| CLAUDE.md Says | Reality |
|---------------|---------|
| `datacraft/` | Does not exist → actual repo is `craftobj/` |
| `computecraft/` (future) | Does not exist → `craftcom/` exists and is complete |
| `parallelcraft/` (future) | Does not exist → was `craftgpu/` (removed/gitignored) |
| 13 crates in craftec-core | 16 crates exist |
| IPC has WebSocket transport | WebSocket NOT implemented in craftec-ipc |

---

## Priority-Ranked Gap List

### 🔴 Critical (Architectural Mismatches)

1. **Ecosystem doc + CLAUDE.md naming mismatch** — `datacraft` vs `craftobj`, `computecraft` vs `craftcom`, `parallelcraft` vs removed
2. **Aggregator architecture** — Design says CraftCOM agent + CraftSQL; impl is native library + P2P gossipsub
3. **craftec-ipc missing WebSocket** — Blocks CraftStudio integration entirely
4. **craftec-identity missing key features** — No social recovery, key rotation, or cross-craft reputation

### 🟠 High (Missing Implementation)

5. **CraftStudio ~15% complete** — Has a full design doc but almost no protocol integration
6. **Settlement SDK missing plan management** — No PricingPlan types, no create/update/delete plan instructions
7. **Settlement SDK missing force_close** — Only cooperative payment channel close
8. **agent-runtime not functional** — WASM sandbox scaffolded but can't actually run agents
9. **4+ repos missing design docs** — craftsec, craftsql, craftvfs, craftcom, scalecraft

### 🟡 Medium (Partial Implementation)

10. **CraftNet aggregator not CraftCOM agent** — Works as native lib, but diverges from arch decision
11. **Reputation computation** — ReputationScore struct exists but no logic
12. **CraftNet mobile (uniffi)** — Crate exists but iOS/Android integration incomplete
13. **Light Protocol double-claim prevention** — Types exist in CraftNet, not fully in core SDK
14. **SP1 distribution-guest** — May need rebuild for current SP1 SDK version

### 🔵 Low (Cosmetic / Minor)

15. **Naming inconsistency** — `fund_creator_pool` vs spec's `deposit`
16. **Segment size not configurable** — Hardcoded 10 MB in CraftOBJ
17. **Windows named pipe** — Path generation exists but server doesn't use it

---

## Recommended Actions

### Immediate (Docs Sync)
1. **Update `docs/CRAFTEC_ECOSYSTEM.md`** to reflect actual repo names and all repos
2. **Update root `CLAUDE.md`** to match reality (craftobj, craftcom, remove future placeholders for things that exist)
3. **Decide on aggregator architecture** — is it CraftCOM+CraftSQL (design) or native+P2P (current)? Update the design doc to match the decision.

### Short-Term (Unblock CraftStudio)
4. **Add WebSocket transport to craftec-ipc** — required for CraftStudio
5. **Add server-sent notifications** — required for real-time UI updates

### Medium-Term (Complete Settlement)
6. **Add PricingPlan system to settlement SDK**
7. **Add force_close instruction**
8. **Add update_config instruction**

### Long-Term (Identity + Agent Runtime)
9. **Implement social recovery + key rotation** in craftec-identity
10. **Make agent-runtime functional** (prerequisite for CraftCOM aggregator architecture)
11. **Write design docs** for craftsec, craftsql, craftvfs, craftcom, scalecraft
