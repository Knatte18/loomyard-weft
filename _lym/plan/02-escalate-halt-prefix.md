# Card 2 — A loom or darn fixer escalate halts with its own prefix and is no hand-back

**Create:**
- `internal/landingshed#EscalateStopPrefix`
- `internal/landingshed#EscalateHalt`
- `internal/battencli#restepSkips`
- `internal/battencli#TestRestepSkips`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#fixerOutcome.haltReason` -> `func (o fixerOutcome) haltReason(row string, escalateJudgedFirst bool) string`
- `internal/landingshed#ParentAlign.align`
- `internal/landingshed#ParentAlign.haltSpent`
- `internal/landingshed#Finalize.runFixer`
- `internal/landingshed#Finalize.gateAndFix`
- `internal/landingshed#ParentAlign.Call`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps`
- `internal/battencli#battenCLI.wire`
- `internal/battenshed/doc.go`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer.binaryRestep`
- `internal/arbiterstop/doc.go`
- `internal/landingshed#TestFixerRun_Outcomes`
- `internal/landingshed#TestHandsBack`
- `internal/landingshed#TestFinalize_GateAndFixer`
- `internal/landingshed#TestMergeIn_Fixer`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/arbiterstop#TestCheck_RefusesLandingHalts`
- `internal/battencli#TestWire_EverySeamFilled`
- `contracts/specs/landing-fixer-report-spec.md`
- `contracts/specs/loom-status-spec.md`
- `contracts/stencils/orch/orch-template-role.md`
- `docs/overview.md`

**Uses:**
- `internal/landingshed#HandsBack`
- `internal/landingshed#LandingStopPrefix`
- `internal/landingshed#parentAlignName`
- `internal/landingshed#finalizeName`
- `internal/landingshed#retiredMergeInName`
- `internal/landingshed#requireHalt`
- `internal/landingshed#newFinalizeGateFixture`
- `internal/arbiterstop#HandsBack`
- `internal/arbiterstop#Check`

**Intent:**
D3: on a loom or darn run a merge fixer's `escalate` at Parent-Align or Finalize halts with a reason opening with the new exported `EscalateStopPrefix`, `"landing escalate: "`, which opens neither with `LandingStopPrefix` nor with `shedadapters.DecisionHaltPrefix`, so `landingshed.HandsBack` and `arbiterstop.HandsBack` stay false for it and `arbiterstop.Check` admits it as an ordinary blocked stop, with no code change in either.

`Deps` gains `EscalateJudgedFirst bool`, told by the caller: true on a loom or darn run, false on a pawl run.
`landingDeps` sets it to `recipe == shedrun.RecipeLoom || recipe == shedrun.RecipeDarn`, the two recipes the record names; the drift guard in `TestLandingDeps_EveryFieldPopulated` builds a loom run, so the field is filled there with no change to its unfilled map.
`fixerOutcome.haltReason` takes `escalateJudgedFirst`: for `fixerEndEscalate` with it true the reason is `EscalateStopPrefix` + the cause as now (row, report path, conflict) + `; way forward: record the deciding decision with` `` `lym loom decision add` `` `in the task worktree, then` `` `lym loom resume` ``;
with it false the reason opens with `LandingStopPrefix`, keeps the cause and its way forward reads `settle the conflict on the task branch, then` `` `lym loom resume` ``, naming no `lym loom decision add`, since that verb refuses a pawl run.
Every other end ignores the flag and keeps its `LandingStopPrefix` reason unchanged.
`ParentAlign.align` and `ParentAlign.haltSpent` pass `m.deps.EscalateJudgedFirst`, and `Finalize.runFixer` passes `fz.deps.EscalateJudgedFirst`.

`EscalateHalt(producer, reason)` is true when producer is `parentAlignName`, `finalizeName` or `retiredMergeInName` and reason opens with `EscalateStopPrefix`; it is the prefix's only reader, and its callers check the blocked state as `HandsBack`'s do.
Batten's binary re-step leaves such a stop alone: the `HandBack` closure in `battenCLI.wire` becomes the package function `restepSkips`, true for a blocked status where `arbiterstop.HandsBack` or `landingshed.EscalateHalt` answers true over its current producer and error, and `wire` tells `restepSkips` as `InnerRun.HandBack`.

Bound (an edge added): only a fixer's `escalate` report changes route, and only on a loom or darn run; every other halt at those rows, and a pawl run's escalate, keep `LandingStopPrefix` and the hand-back.
A run halted under `LandingStopPrefix` by an older build stays a hand-back, with no migration.

Tests:
- `TestFixerRun_Outcomes` calls `haltReason(fixerTestRow, false)` for its existing assertions, and the escalate row's fragments name `settle the conflict on the task branch` and no `lym loom decision add`; each row also reads `haltReason(fixerTestRow, true)`: for the escalate end it opens with `EscalateStopPrefix` and holds the report path, `fixerTestRow`, `decision D3`, `` `lym loom decision add` `` and `` `lym loom resume` ``, and for every other end it equals the false reason.
- `TestHandsBack` gains an `escalate` column read by a second assertion on `EscalateHalt`: rows with `EscalateStopPrefix` at Parent-Align, Merge-In and Finalize answer `HandsBack` false and `EscalateHalt` true, one at Describe answers both false, and every existing row answers `EscalateHalt` false.
- `TestFinalize_GateAndFixer`'s escalate subtest gains a case with `EscalateJudgedFirst` true on the fixture's Finalize deps: the call is a halting Stuck whose reason opens with `EscalateStopPrefix`, names the report path, and `HandsBack` reads false and `EscalateHalt` true for it; the existing case, with the field false, keeps `requireHalt` and names no `lym loom decision add`.
- `TestMergeIn_Fixer`'s first-run escalate subtest gains the same judged-first case at Parent-Align.
- `TestWire_BeamRecipe`'s table gains `wantEscalateJudgedFirst`: true for loom and darn, false for pawl.
- `TestCheck_RefusesLandingHalts` gains rows for an `EscalateStopPrefix` halt at Finalize and at Parent-Align: Check admits each, and `HandsBack` reads false.
- `TestRestepSkips` (tier0, table-driven) pins `restepSkips`: a blocked `LandingStopPrefix` halt at Finalize, a blocked decision halt, and a blocked `EscalateStopPrefix` halt at Finalize and at Parent-Align answer true; a failed status carrying the escalate prefix, a blocked escalate-prefixed reason at Describe and a blocked plain halt answer false.
- `TestWire_EverySeamFilled` asserts that the wired `c.env.InnerRun.HandBack` answers true for a blocked status at Finalize whose error opens with `EscalateStopPrefix`, so the wiring of `restepSkips` is pinned beside the function.

Docs: `internal/landingshed/doc.go`'s escalate and HandsBack paragraphs name the prefix, the predicate, the pawl case and the bound.
`Finalize.gateAndFix`'s doc and `ParentAlign.Call`'s doc say a loom or darn escalate halts with `EscalateStopPrefix`, and every other halt of theirs with `LandingStopPrefix`.
`internal/battenshed/doc.go`'s binary re-step paragraph says an escalate halt `InnerRunDeps.HandBack` marks is left alone too.
The `InnerRunDeps.HandBack` field doc and `binaryRestep`'s doc name a landing halt, a decision halt or a loom or darn escalate halt as what the seam marks and the re-step leaves alone.
`internal/arbiterstop/doc.go` says a loom or darn escalate halt is no landing halt and is eligible as a blocked stop.
The landing fixer report spec says an `escalate` goes to the arbiter first on a loom or darn run and to the orch on a pawl run.
The loom status spec's `hand_back` section says a loom or darn fixer escalate halt opens with the escalate prefix, is not marked `hand_back`, and goes to the arbiter first.
The orch role stencil's `Landing halts` says a loom or darn escalate reaches the orch only as the arbiter's hand-on and a pawl one as a landing halt, and its landing-stop bullet keeps "escalated a decision" for a pawl run only.
`docs/overview.md`'s landing-halt line says the same.

**ImpactSummary:** `haltReason`'s new flag reaches its three in-package callers and its test; the new `Deps` field is set only by `landingDeps`; batten's re-step skip moves into one tested function, and `arbiterstop` changes only in doc and test rows.
