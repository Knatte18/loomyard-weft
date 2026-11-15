# Plan round 1 — fixer report

`lym loom validate-plan` reports ok after every change below.

## Changes

- **F1 (BLOCKING), fixed as proposed.** Card 5 creates `plan:internal/pairteardown#runShedRow` in `followup.go`'s const block beside `landRow` and `worktreeTeardownRow`, and lists `followup.go` under Edit for that block's doc.
  The Intent states that `pairteardown` cannot import `battenrecipe` and names no `battenrecipe` symbol: `BattenStarted` reads the three local rows, and `ChildLanding` and `PastParentMerge` read `landRow` and `worktreeTeardownRow`.
  A `TestReadRun` row pins `runShedRow` to `battenrecipe.NameRunShed` and pins the started set to the recipe's rows less `RowsBeforeRunShed()`.
  The test-only import closes no cycle.
- **F2 (BLOCKING), fixed as proposed.** Card 11 lists `dobbyengineAllowedImports` under Edit, and its Intent says the list gains `internal/runfsm`.
- **F3, fixed as proposed.** Card 6 adds `TestBattenIntegration_Rows`, the `battenCLI` doc, `RunStatus`, `runStatusSeparator` and `orch-role.md` under Edit, and `boardengine/doc.go` under Uses.
  It extends the Intent, the tests and the ImpactSummary to match.
- **F4, fixed differently.**
  - Card 4 creates `FetchedTaskBranchOnOrigin`, which uses `TaskBranchOnOrigin`'s matching rule over the fetched refs, read in-process as `TaskBranchSlugs` does.
  - It states the bound: a branch deleted since the last fetch reads present until the next fetch.
  - Card 5 fills `TaskBranchOnOrigin` through the new read and lists it under Uses.
  - The proposed `TestReadRun` row "showing the read makes no network call" was not added. `TestReadRun` is tier0 over fakes, so a fake seam cannot show the absence of a network call.
  - That property is pinned instead in card 4's tier1 `TestOriginRunReads`: a row repoints origin's URL at a missing path after the fetch and still reads the pushed branch.
- **F6, fixed as proposed, then extended by the self-check.**
  - Card 8 has a cancel lock that another process holds read as present with a live writer, whether or not the record holds an intent yet. `judgeCancelMarker` takes that reading.
  - The refusal for a live writer that has no intent yet names no start time.
  - `TestCancelMarker` and `TestJudgeCancelMarker` gain the held-lock-with-no-intent row.
- **F7, fixed as proposed.**
  - Card 3's `Verdict` gains `PausedByIntent`, set by `Judge`.
  - The paused refusal names `lym tender resume <slug>` then the re-run for an intent, and keeps `lym batten run <slug>` or `lym loom resume` for a shed pause.
  - `TestAct` gains a row for each wording.
  - The overview's shed-pause-aside rationale is updated to the two wordings.
- **F8, fixed differently.**
  - Card 11 lists `drivable` under Edit and drops its paused clause, so `drivable` reads only running.
  - The proposal also drops `RunView.ReachedParent`, `DobbyRuns`' reached-parent read with `unreadableReachedParent`, and rewrites `TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent`. Those parts were not made.
  - The reason: `strandedEnded` reads `drivable` (`internal/dobbyengine/ended.go:36`). Dropping the clause and the field would read a paused run whose work reached its parent as stranded once its entry ended, and the dobby would tear down a pair whose Land is unfinished. `TestStrandedEnded`'s `PausedWithItsWorkOnItsParent` row (`ended_test.go:28`) pins that it is not stranded.
  - `strandedEnded` therefore keeps the clause, and the field and its read stay for it.
  - `TestPlanPass`'s paused-with-work rows give no pass.
  - `TestDrivePasses_ExitHandling` is listed under Edit, and its lift subtest for such a run (`passes_test.go:539`) asserts no pass.
- **F9, fixed as proposed.** In card 3, `Derive` walks `Order()`, so production reaches `Order`.
- **F10, fixed as proposed.** Card 11's `doc.go` bullet covers every paragraph that names the marker. Card 7's Long-text change says that the end-only cancel leaves its intent committed locally.
- **F11, fixed as proposed.** In card 14, `Resume` treats a terminal wait as act. `TestResume` gains a landed-lifts row.
- **F12, fixed as proposed.** Card 7 names the act-rule refusal's input as a board `done` slug with no batten record, read `Landed`, with the terminal wording. The hand-over row stays as an extra case, in the Intent and in `TestCancelRun`.
- **F13, fixed as proposed.** Card 9 drops the nil-`PairStands` branch, and every test-built `cleanupDeps` that reaches a cancelled record sets `PairStands`.
- **F14, fixed as proposed, then corrected by the self-check.** Card 9's Intent and ImpactSummary say that every `Teardown.Run` caller reaches the deletion.
- **F15, fixed as proposed (the second option).** Card 12's Intent states why `lym loom run` and `step` are exempt and what bounds that. The ImpactSummary points to that reason.

## Self-check

- fix-consistency, MEDIUM: `DobbyRuns` and `pairteardown`'s `cancelling` still read the intent alone, despite F6.
  Disposition: fixed as proposed. In card 8, `DobbyRuns` reads through `readCancelMarker`, and `pairteardown`'s `cancelling` also reads a cancel lock that another process holds. Card 11's `RunView.CancelMarker` doc follows.
- fix-consistency, LOW: the stranded-run docs (`ended.go:31-32`, `doc.go:285`) still give the no-pass reason. Merged with the affected-set LOW on the same texts and on `drivable`'s doc.
  Disposition: fixed as proposed. Card 11 rewords the `drivable` doc, the `strandedEnded` doc and `doc.go`'s stranded-run paragraph.
- fix-consistency, LOW: the overview's fabric-origin-reads decision and the Card Index still say two reads.
  Disposition: fixed as proposed. They now say three reads and name the task-branch read.
- fix-consistency, LOW: no Shared Decision records card 12's exemption.
  Disposition: fixed as proposed. A new Shared Decision, verbs-that-do-not-ask, covers `lym batten goto` (card 10) and `lym loom run` and `step` (card 12).
- fix-affected-set, LOW: cancel-marker prose survives in `refuseBeamCancel`'s doc, `planPass`'s doc, `clearRunPhase`'s doc and `registeredBattenAlive`'s doc.
  Disposition: fixed as proposed. Card 8 lists `refuseBeamCancel` and `clearRunPhase` under Edit and rewords those docs and `registeredBattenAlive`'s doc. Card 11 rewords `planPass`'s doc.
- fix-claims, MEDIUM: card 9 omits a third `Teardown.Run` caller, the beam fork's member removal (`internal/battencli/beamfork.go:176-180`).
  Disposition: fixed as proposed. Card 9's Intent and ImpactSummary name it.
- fix-proposal, BLOCKING: trying the shared lock on every read fails when the scratch root is missing, because `TryAcquireReadLock` does not create the parent directory, and it leaves a lock file behind per slug.
  Disposition: fixed as proposed. In card 8, an absent lock file, scratch root included, reads free, and the read creates nothing. `TestCancelMarker` gains a row for a prime with no scratch root.

## Disputed

None.

## Deferred

None.

## Rulings

- **F5 (decision).** The parent recorded the ruling with `lym loom decision add` as the decision record's Added after Discussion entry "A shed pause is an aside of the act rule" (parent, 2026-11-15). It reached this session as the LOOM decision notice #1345.
  The ruling licenses the shed-pause aside for `lym batten run`, `lym batten step`, `lym loom start`, `lym loom resume`, `lym batten cancel` and `lym tender resume`.
  Change made: the overview's shed-pause-aside Shared Decision cites that entry. Cards 3, 7, 10, 12 and 14 keep `ShedPause` in `runfsm.Aside` as they had it.
  The parent was not told, since the ruling was neither mistaken nor redundant.
