```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every path that writes or reads the state D1–D4 change.

**Paths traced, per decision**

- **D3, `landingshed.HandsBack` → `shedverbs.Spec.handsBack`** (loop envelope `hand_back`, `internal/shedverbs/loop.go:335`; `lym shed status`/`lym loom status` `hand_back`, `internal/shedverbs/status.go:187`): survives. The escalate reads as not `hand_back`, which is what D3 intends.
- **D3, Go driver `HandBack` rule row** (`internal/sheddriver/driverules.go:218`), including status-built stops through `Deps.HandsBack` (`internal/loomcli/drivewiring.go:302`): survives. The escalate falls through to the later rows.
- **D3, `arbiterstop.Check` `LandingHalt`/`DecisionHalt`** (`internal/arbiterstop/arbiterstop.go:164`), read by `shedarbiter` `settleFresh` (`internal/shedarbiter/call.go:180`): survives. The escalate is eligible as a blocked stop.
- **D3, batten binary re-step `HandBack` seam** (`internal/battencli/wire.go:1067` → `internal/battenshed/innerrun_binaryrestep.go:63`): survives only through D3's named skip.
- **D3, loom `Spec.HandBack` arming** (`internal/loomcli/arm.go:309`): survives. It is the same predicate.
- **D3, LLM driver session** (seed `driver: llm`, `internal/shedrun/seed.go:46`; prompt `internal/loomcli/driverprompt.go:20`; stencil `contracts/stencils/shed/shed-template-driver.md:98,101,145`; guide `shed-template-driver-guide.md:67-68`): **breaks**. See F1.
- **D3, `contracts/specs/loom-status-spec.md:96-97`**, which defines the `hand_back` key for loom and darn: **misreads**. See F2.
- **D3, orch role stencil line 128** ("A landing halt reaches you with the status marked `hand_back`"): covered by D5's `Landing halts` rewrite. It holds only if that line is in the rewritten span; D5 does not name it.
- **D4, Finalize parent lock across the arbiter's turn** (`internal/landingshed/finalize.go:198,265`): survives. The lock is released on every way out, the halt included, so the arbiter's turn holds no lock.
- **D4, arbiter resume goto at the halted row through `GotoGuard`/`finalizeGuard`** (`internal/loomcli/finalizeguard.go:72`): survives unchanged. It is the same guard the orch's goto meets today.
- **D1/D2, merge record `Overlap` read by the fixer** (`internal/landingshed/fixerrun.go` `FixerInputs.MergeRecordPath`): survives, as the Auto-mode assumption keeps it.

**Findings**

**F1. MEDIUM, design.** Location: D3 (and D5).
- **Summary:** D3 removes the `hand_back` mark from a loom or darn escalate halt, but the LLM driver session's re-launch rules read only that mark. A run seeded `driver: llm` would arm the binary watch after the arbiter's hand-on and re-launch the row over the undecided collision on a new build. That is the exact re-step D3 closes for batten, and the record is silent on it. D5 also does not list the driver stencil or its guide.
- **Evidence:**
  - `contracts/stencils/shed/shed-template-driver.md:98`: re-launch at a blocked stop "not marked `hand_back`" when `binary_changed`.
  - Same file, line 145: start the binary watch at a hand-back "not marked `hand_back`".
  - `contracts/stencils/shed/shed-template-driver-guide.md:67-68`.
  - `internal/shedrun/seed.go:44-50`: `DriverLLM` is a legal loom driver.
  - `internal/loomcli/driverprompt.go:20`: loom fills this stencil.

**F2. LOW, consistency.** Location: D5.
- **Summary:** `contracts/specs/loom-status-spec.md`'s `hand_back` section still says that, for loom and darn, a stop at Parent-Align or Finalize opening with the landing-stop prefix is a hand-back. The section also defines what the key means for drivers. D5's docs list does not name this spec, so the status contract would go stale under D3.
- **Evidence:** `contracts/specs/loom-status-spec.md:96-97`.

**Tool calls**

1. grep for every Go reader of `LandingStopPrefix`, `HandsBack` and `hand_back`: the manifest held the cited readers, not a full sweep.
2. grep for prose readers of `hand_back`, Finalize's halt and lock release, and decision add's commit: stencils, specs and the halt path were not in context.
3. grep for agent-driver stencil use and Finalize lock release: needed to know whether `shed-template-driver` is live for loom.
4. read `driverprompt.go`, the seed driver vocabulary, the driver stencil's re-launch and arbiter rules: to confirm the LLM driver is a live loom driver and how it reads `hand_back`.
5. read `loom-status-spec.md` `hand_back`, `seed.go` `DriverLLM`, the driver stencil hand-back steps: for the exact wording behind F1 and F2.
