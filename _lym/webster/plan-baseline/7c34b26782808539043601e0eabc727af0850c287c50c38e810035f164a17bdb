# Card 16 — drift-verify-lines

**Edit:**
- `internal/driftengine#DriftVerify`
- `internal/driftengine#NewDriftVerify`
- `internal/driftengine#DriftVerify.Call`
- `internal/driftengine#DriftVerify.pass`
- `internal/driftengine#DriftVerify.probeInterruptedAgent`
- `internal/driftengine#DriftVerify.tableFor`
- `internal/driftengine#DriftVerify.runAgent`
- `internal/driftengine#DriftVerify.now`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine#verifyFixture`
- `internal/driftengine#newVerifyFixture`
- `internal/driftengine#verifyFixture.call`
- `internal/driftengine#verifyFixture.scope`
- `internal/driftengine#verifyFixture.commit`
- `internal/driftengine#passingCompare`
- `internal/driftengine#failingCompare`
- `internal/driftengine#requireStuck`
- `internal/driftengine#requireDone`
- `internal/driftengine/doc.go`
- `internal/shedrecipe#driftVerifyEntry`
- `internal/shedrecipe#TestDriftEntries`
- `internal/shedrecipe#driftTestEnv`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe/doc.go`
- `internal/loomrecipe/doc.go`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/shedbuild#engineMinimalConfig`
- `internal/loomcli#driftDeps`
- `internal/loomcli#driftTables`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#driftDeps`
- `internal/battencli#intentionallyNil`
- `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestDriftRows_EntryGate`

**Create:**
- `plan:internal/driftengine#darnWriteRow` -> `const darnWriteRow = "Darn-Write"`
- `plan:internal/driftengine#websterRow` -> `const websterRow = "Webster"`

**Delete:**
- `internal/loomrecipe#driftRecheckType`
- `internal/driftengine#DriftVerify.keepAcceptedProfile`
- `internal/driftengine#newestCompareLog`
- `internal/driftengine#DriftVerify.resumedFrom`
- `internal/driftengine#DriftVerify.updateScope`
- `internal/driftengine#DriftVerify.recordCompare`
- `internal/driftengine#DriftVerify.markGateHalt`
- `internal/driftengine#DriftVerify.guardCommits`
- `internal/driftengine#DriftVerify.keepFindings`
- `internal/driftengine#DriftVerify.halt`
- `internal/driftengine#DriftVerify.haltReason`
- `internal/driftengine#DriftVerify.haltFrom`
- `internal/driftengine#DriftVerify.stuckAfterAgent`
- `internal/driftengine#DriftVerify.dirtyEntry`
- `internal/driftengine#DriftVerify.verifyEntry`
- `internal/driftengine#DriftVerify.onceCommitted`
- `internal/driftengine#DriftVerify.compareEntry`
- `internal/driftengine#recordingSeats`
- `internal/driftengine#recordingSeats.Probe`
- `internal/driftengine#recordingSeats.Resume`
- `internal/driftengine#recordingSeats.Run`
- `internal/driftengine#recordingSeats.failedGate`
- `internal/driftengine#dirtyHaltOpening`
- `internal/driftengine#gateHaltOpening`
- `internal/driftengine#compareHaltOpening`
- `internal/driftengine#generalHaltOpening`
- `internal/driftengine#dirtyGateName`
- `internal/driftengine#compareGateName`
- `internal/driftengine#dirtyGateAttempts`

**Extends:**
- `internal/driftengine#Deps`

**Uses:**
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomcli#conflictSessionStopper`
- `plan:internal/driftengine#Check`
- `plan:internal/driftengine#WriteRecords`
- `plan:internal/driftengine#ReadRecords`
- `plan:internal/driftengine#WriteJudgeRequest`
- `plan:internal/driftengine#TakeJudgeSnapshot`
- `plan:internal/driftengine#ReadJudgeSnapshot`
- `plan:internal/driftengine#JudgeSnapshot.Changes`
- `plan:internal/driftengine#ParseJudgement`
- `plan:internal/driftengine#CheckBaseline`
- `plan:internal/driftengine#DeferHaltPrefix`
- `internal/driftengine#gateLoopSeats`
- `internal/driftengine#fakeChair`
- `plan:internal/driftengine#commitStore`
- `plan:internal/driftengine#commitStore.deps`

**Intent:** Rewrite Drift-Verify as containment by code and the judge under `lines` mode (D4, D6, D7, D9, D10), and run Drift-Recheck on the same engine.
`command` mode is card 17; until then a recorded `command` mode that step 2 below leaves standing, that is with scope other than `off`, no deferral and no waiver, is a Stuck naming `lym drift defer`, checked right after step 2.

`Deps` gains `EndRoles func(roles []string) error`.
The route that fixes code, which every regression Stuck names, is derived by comparing `Deps.Rows` against this card's row constants `darnWriteRow` and `websterRow`, never repeated literals: `lym loom goto --to Darn-Write then lym loom resume` when the rows hold `Darn-Write`, else `lym loom goto --to Webster then lym loom resume`, whose Merriam, with every card done, still starts and meets the cells in its run-level `drift` entry and hands them to the verify fixer (cards 10, 11; Shared Decisions, with the researched re-entry fact).
`EndRoles` is wired from reed's strand table and the runner's strand stop, the two seams `conflictSessionStopper` is built over, stopping every live strand owned by this worktree whose role is exactly one of `roles`; it is told only the removed roles, so no other chair is ended (Shared Decisions); `envkit` fakes it and `battencli`'s `intentionallyNil` lists it.

`DriftVerify.Call`, after `passWhenSwitchedOff`, runs in order:
1. `EndRoles` of `drift-baseline` and `drift-verify`, the removed agents' roles, before any work (D4).
2. `ReadRecords`: no scope record is Done; a records mismatch is a Stuck naming the parent's `lym drift amend <slug> --restore` (D6); scope `off` (naming its source), a deferred capture and a `WaivedAt` waiver are Done (D3, D12); a record with no `Diff` records config's `diff` and `key_patterns` through `WriteRecords` and keeps them (D3).
3. A scope record with no `awaited_at` is a Stuck saying the capture never counted and naming `lym drift defer`; otherwise `CheckBaseline`'s fault, a missing baseline directory or a changed digest, is a Stuck naming `lym drift defer`; whether the capture counted is read from the tracked record, never `capture.json` (D7, D12; Shared Decisions).
4. A warp HEAD equal to `LastPassHead` with `LastPassAmendments` equal to the amendment count is Done (D4).
5. An open `judge_session` mark resumes the judge.
   A recorded `SnapshotLost` takes a new snapshot first, then resumes or respawns.
   A gone snapshot with no `SnapshotLost` recorded records it and is a Stuck naming it, whose way forward is the parent or operator checking both checkouts for the judge's edits, then `lym loom resume`.
   Otherwise the trees are first compared with the recorded snapshot, as on the session's return in step 10, and a change is the same Stuck; only once they match does the probe resume a live chair with the recorded spawn id, or respawn under the same spawn id and the same recorded snapshot, never a new one (D9).
6. A run past Drift-Expect with no seal commit and no `expected.yaml` records `Unsealed` once (D6); a run with an `expected.yaml` and no seal commit is left unmarked, so `CheckSeal` faults on it naming `--discard`.
7. `Check`: a seal fault is a Stuck naming its way forward (D6); a failed current capture, a step refusal among them, is a Stuck naming its reason, its log and three ways forward, `lym loom resume`, the fix route, and the parent's `lym drift defer` (D7); any unexpected cell is a Stuck naming each with case, identity, old and new value, the parent's `lym drift amend <slug>` and the fix route (D7).
8. Each expected cell whose exact tuple has a stored ruling reuses it; a reproduced stored `does not match` is a Stuck naming the cells, reasons, `lym drift amend` and the fix route, with no judge (D10).
9. With no cell left to judge, the check passed: `LastCheck`, `LastPassHead` and `LastPassAmendments` are recorded and the row is Done.
10. Otherwise one judge: Drift-Verify removes any `judgement.yaml`, writes the `JudgeRequest` under a fresh spawn id, opens the `judge_session` mark through `WriteRecords`, takes the snapshot, and runs the judge through `shedadapters.NewMultiLLMProducerSourced` with its table gated by the row's `judgement` entry and its findings kept (`KeepGateFindings`).
    On the session's return, before closing the mark: a change `Changes` names is a Stuck naming the changed paths, the moved HEADs and the snapshot's directory, whose way forward is resetting a moved branch, restoring a changed tracked file from its HEAD or an untracked path from the snapshot's copy, deleting an added path and the changed scratch directory, then `lym loom resume`; a baseline digest mismatch is a Stuck naming `lym drift defer`.
    The rulings are stored as tuples in `judged.json` with the judge's strand and time, and each listed open range as a `JudgedRange` with the covering entry's `why` and the judge's sentence, replacing an earlier listing of the same range; the mark is closed and `LastCheck` recorded, all in one `WriteRecords`, and `judgement.yaml` is removed in the same step, on a Done and a `does not match` Stuck alike, since the stored tuples are the record (Added after Discussion); any `does not match` is a Stuck naming the cells, reasons, `lym drift amend` and the fix route; all `matches` records the pass and is Done.
    An exhausted `judgement` entry is the shed's Stuck, whose resume respawns the judge.
Every Stuck naming a parent verb, and the judge-change and snapshot-lost Stucks, whose ways forward are the parent's or operator's, open with `shedadapters.DecisionHaltPrefix`; one naming `lym drift defer` carries `DeferHaltPrefix` right after it (Shared Decisions).
No agent stands between a finding and the stop chain; the judge edits no code, expectation or record.

`driftVerifyEntry` reads `subject` and a `gates` list of exactly one entry named `judgement`, refusing `gate_key` and every other key, and checks the seams `Config`, `Status`, `Head`, `CommitWithTrailers`, `TrailerCommit`, `FileAt`, `CaptureCurrent`, `JudgeTable`, `Checkouts`, `WeftChangedSince`, `EndRoles` and `Compare`; `VerifyTable`, `CompareAccepted` and `DirtyPaths` are no longer required.
Both recipes' Drift-Verify config becomes `{subject: decision-record, gates: [{name: judgement, attempts: 3}]}` with no `gate_key`, and Drift-Recheck becomes `engine: DriftVerify` with the same config and no `on_stuck`, so its Stuck goes through the stop chain (D4); the headers and row comments describe containment and the judge, and drop the agent that accepts and fixes drift.
The `loomrecipe` package doc drops the sentence that Drift-Recheck's `on_stuck` names Drift-Verify and says Drift-Recheck runs the DriftVerify engine.
`minimalEntries`' DriftVerify row becomes `gates: [{name: judgement, attempts: 3}]`, since the entry now refuses a `verify` gate.
The rewritten `verifyFixture` leaves `captureForAcceptance`, `agentAccept` and `sessionAccept` in place untouched; card 25 deletes them with their last callers.
`engineMinimalConfig`'s DriftVerify config, the producer tables, whose Drift-Recheck rows now read `driftVerifyType` so `driftRecheckType` goes, darn routing, the progress order's Drift-Recheck edge, the entry-gate test's Drift-Recheck engine, the recipe engine set and `TestTapestryMotifBlock`'s fixture `tapestry-rows-before-motif.yaml` follow.

`TestDriftVerify` (tier0, the rewritten `verifyFixture` over `commitStore`, fakes for the other seams, `gateLoopSeats` and `fakeChair`, `t.Parallel`) covers, as rows: an unexpected cell stops naming case, identity, old and new value, `lym drift amend` and, with darn's rows, `lym loom goto --to Darn-Write then lym loom resume`, and with tapestry's rows `lym loom goto --to Webster then lym loom resume`; every cell expected with stored `matches` is Done with no judge; a cell with no stored ruling spawns one judge whose request lists it; an `added case` cell's request names the case's file in the current capture; a reproduced tuple reuses its ruling and a changed new value is judged again; a stored `does not match` stops without a judge; a judge `does not match` stops naming the cell; `judgement.yaml` is gone after the judge's all-`matches` Done and after its `does not match` Stuck; a judge's open range is stored with its `why` and sentence, and a later judge listing the same range replaces it; a stale `judgement.yaml` is removed before the spawn; an open mark with a live chair resumes it under the recorded spawn id and with none respawns under it; a judge edit of a tracked file, of `scope.json`, an added untracked file or a commit on either side stops naming the paths, a change made before the spawn does not, and a resume with the edit still there stops again against the recorded snapshot without spawning the judge; an open judge mark resumed after loom committed the row's status on the weft resumes the judge and does not stop, and so does a resume after a judge-change Stuck whose trees were restored, with loom's halt note committed in the friction directory; a baseline edit made during the judge session stops naming `lym drift defer`; a gone snapshot with an open mark stops and its resume takes a new one; a records mismatch stops naming `--restore`; a scope record whose recorded diff is `command` is a Stuck naming `lym drift defer`, its reason opening with `DecisionHaltPrefix` then `DeferHaltPrefix`, with no judge spawned and no compare run, and the same record once deferred is Done; a record with no `awaited_at` stops saying the capture never counted and naming `lym drift defer`; an edited baseline file before any judge, and a missing baseline directory with no `capture.json`, each stop naming `lym drift defer`; an `expected.yaml` with no seal commit past Drift-Expect stops naming `--discard`, and with neither file nor seal the run records `Unsealed` and checks against an empty expectation; scope `off`, a deferral and a waiver are Done; the head of the last pass with no amendment since is Done; a record with no `Diff` records config's mode and patterns, and a later config change of `key_patterns` changes neither; a capture failure names its log, `lym loom resume`, the fix route and `lym drift defer`; a `judgement` entry that fails every attempt ends the call Stuck and the next call respawns the judge; each Stuck naming a parent verb or a checkout restore opens with `DecisionHaltPrefix`, and each naming `lym drift defer` carries `DeferHaltPrefix` right after it; a live chair of the removed `drift-verify` role is ended on entry; Drift-Recheck's row runs the same pass.
`TestDriftEntries` covers the `judgement` gate, the refused `gate_key` and a missing `WeftChangedSince` seam, and drops its `VerifyNilCompareAccepted` row, since the entry no longer requires `CompareAccepted`.

**ImpactSummary:** Replaces Drift-Verify's body and its private helpers inside driftengine, switches Drift-Recheck's engine in both recipes, adds two Deps fields wired in loomcli and the `darnWriteRow` and `websterRow` row constants in driftengine; the old Recheck engine stays registered but unused until card 23.
