# Card 10 — drift-containment-check

**Create:**
- `internal/driftengine#ClassifiedCell`
- `internal/driftengine#CheckReport`
- `internal/driftengine#Check`
- `internal/driftengine#CheckReport.Findings`
- `internal/driftengine#NewDriftGate`
- `internal/driftengine#TestCheck`

**Edit:**
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsTheLegalValues`
- `internal/shedrecipe/doc.go`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`
- `internal/loomrecipe#TestDarnVerifyBudget`
- `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`
- `internal/shedrecipe#TestWebsterEntry_GateKeys`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `internal/driftengine#ReadRecords`
- `internal/driftengine#CheckBaseline`
- `internal/driftengine#CheckSeal`
- `internal/driftengine#PastRow`
- `internal/driftengine#expectRow`
- `internal/driftengine#ParseExpectation`
- `internal/driftengine#Expectation.Merge`
- `internal/driftengine#Expectation.Cover`
- `internal/driftengine#Expectation.Uncovering`
- `internal/driftengine#ListCaptures`
- `internal/driftengine#DiffCaptures`
- `internal/driftengine#CompileKeyPatterns`
- `internal/shuttleengine#GateEntry`
- `internal/driftengine#commitStore`
- `internal/driftengine#commitStore.deps`

**Intent:** D7's containment by code as one function every check calls, and D8's `drift` gate entry over it on the two writers.

`Check` writes no tracked record and decides nothing; Drift-Verify (card 16) maps its report to outcomes.
It runs, in order, stopping at the first that ends the check:
1. `ReadRecords`; no scope record, scope `off`, a deferred capture and a `WaivedAt` waiver each set `Skip` with the reason (D3, D12).
2. The mode and patterns: the recorded `Diff` and `KeyPatterns`, or config's when the record has none yet (D3).
3. A current row not past Drift-Expect by `PastRow(d.Rows, status row, expectRow)` sets `Skip` ("no expectation is sealed yet") and ends the check (D6), before any baseline check, since a real run has no `awaited_at` before Drift-Await counts the capture.
4. A scope record with no `awaited_at` sets `BaselineFault` ("the capture never counted") naming `lym drift defer`; otherwise `CheckBaseline`'s fault, a missing baseline directory or a changed digest, sets it; either ends the check (D7, D12). Whether the capture counted is read from the tracked record alone, never `capture.json` (Shared Decisions).
5. A records mismatch sets `RecordsFault` naming the parent's `lym drift amend <slug> --restore`; the check goes on over the committed copies (D6).
6. `CheckSeal` against the capture record's head: `NotYet` sets `Skip` ("no expectation is sealed yet"); a fault sets `SealFault` and the check goes on against `Sealed`, or an empty expectation when there is none.
   The sealed bytes and every `entries` amendment are parsed by `ParseExpectation` under the mode against `ListCaptures` of the baseline, and merged.
7. Under `lines`: `CaptureCurrent`; a failed capture sets `CaptureFault` naming its log and ends the check.
   `DiffCaptures` against the current directory, leaving out the scope record's `Skip` cases when its scope is the old `partial` (D3), then each cell is classified by `Cover` into a `ClassifiedCell` (`Cell`, `Coverage`, `Expected`); `Uncovering` lists the entries with no cell.
8. Under `command`: `Deps.Compare` runs the declared compare against the baseline; the report carries its exit, log and pass.

`CheckReport.Findings` renders the report: one line per unexpected cell naming case, identity (key with `n` when set, or `#line`), old and new value, and each fault with its way forward.
`NewDriftGate` returns the gate the `drift` entry runs: it passes on `Skip`, on a baseline, records or seal fault, which no writer session can clear (the findings still carry them, logged at Info), and under `command` whatever the compare's exit, reporting it (D2, D8); it fails on a `CaptureFault` naming the log, with the way forward: fix the code when the change broke the capture, otherwise leave it, since the stop that follows goes to the parent, whose way forward is a fix of the environment the capture names, then `lym loom resume`; and it fails on any unexpected cell under `lines`, with the findings and the way forward: fix the code so the value returns to its baseline, or, when the change is intended, leave it for the parent's `lym drift amend` at the Stuck that follows.

`resolvePlainGateClosure` gains the name `drift`, requiring the `Env.Drift` seams `Config`, `Status`, `CaptureCurrent`, `TrailerCommit`, `FileAt` and `Compare`, and returns `NewDriftGate` over `driftDepsFrom(env)`; the default refusal lists `drift` among the legal names.
The darn recipe's Darn-Write `gates` gains `{name: drift, attempts: 3}` after `verify`, with no fixer (D8); the tapestry recipe's Webster row gains `gates: [{name: drift, attempts: 3}]`, which `websterEntry` already passes to Merriam's run-level gate.
`gatedRowFanViolations` admits the `Implementer` engine as a gated row: Merriam already runs its run-level `verify` gate at turn ends while card forks are spawned, so the recipe-level entry rests on the same turn-end rule; its hand-authored violation case moves to a `Bouncer` row.
Bound: the gate entry writes only its scratch capture directory, and Drift-Verify repeats the check as the guarantee (D8).
The card and batch gate keys get no drift check (Shared Decisions).

`TestCheck` (tier0, on `commitStore` with fakes for the other seams, `t.Parallel`) covers: an unexpected cell is listed with case, identity, old and new value; every cell expected yields no unexpected cell; an expected entry with no cell is in `Uncovering`; scope `off`, a deferral and a waiver skip with no capture; a row before Drift-Expect, with `awaited_at` unset, skips saying no expectation is sealed yet and reports no `BaselineFault`; a record with no `awaited_at` at a row past Drift-Expect reports `BaselineFault` saying the capture never counted; a missing baseline directory with `awaited_at` set and no `capture.json` reports `BaselineFault` naming `lym drift defer` and runs no capture; a records mismatch reports `--restore` and classifies against the committed amendment; a seal fault classifies against the sealed copy, under `lines` and under `command`; a failed current capture names its log; under `command` the compare runs and the capture does not, and a failing compare passes the gate reporting exit and log; an old `partial` record with a skip list leaves its cases out of the diff; the recorded `diff` wins over a changed config, and a record with no `Diff` reads config's mode and patterns.
Gate rows: the gate fails on an unexpected cell and on a `CaptureFault`, and passes on scope `off`, a deferral, and each of a `BaselineFault`, a `RecordsFault` and a `SealFault`, its findings still naming the fault and its way forward.
The shedrecipe tests gain the `drift` row and legal name, `newTestEnv` fills the seams, `TestWebsterEntry_GateKeys` gains a row where the Webster row's `gates: [{name: drift}]` reaches the producer's run-level gate, `TestDarnVerifyBudget` expects `drift: 3` on Darn-Write, `TestTapestryMotifBlock`'s fixture `tapestry-rows-before-motif.yaml` gains the Webster row's `gates`, and the recipe comments describe the entry.

**ImpactSummary:** Adds the `drift` gate name to the shared resolver and one entry to Darn-Write and to Webster's run-level gate; the fork-rule test admits the Implementer row on the record's D8.
