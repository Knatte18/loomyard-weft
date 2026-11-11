```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, listing for each behaviour the test that pins it, or none.

**Per behaviour: pinning test**

- D1 caller spawns a worker and runs no step: `TestRunAdopt`, rows "OrchStrandSpawns…" and "DoneEndsWith…".
- D1 live worker followed, nothing spawned: `TestRunAdopt` "LiveRecordOfTheSameSubjectsIsFollowedWithoutSpawning".
- D1 other subjects refused: `TestRunAdopt` "UnfinishedRecordOfOtherSubjectsIsRefusedWithoutSpawning", and `TestRunWorker` "OtherSubjectsAreRefusedNamingTheRecordedAdopt" for a direct worker call.
- D1 exit codes for done, stopped and interrupted: `TestRunAdopt` and `TestFollow`.
- D1 `still running` at the cap: `TestRunAdopt` "AgentStrandFollowsToTheKeepWarmCap…" and `TestFollow` "AtTheCap…".
- D1 plain shell follows uncapped and never asks keep-warm: `TestRunAdopt` "EmptyStrandFollowsUncapped…" and `TestFollow` "ACapOfZero…".
- D1 follow wake backoff: `TestFollow` "AnUnchangedRecordDoubles…".
- D1 pid and start-ticks ownership, and the later-start rule:
  - `TestFollow` "WorkerThatLostTheLock…SameSubjectsFollowsIt".
  - `TestFollow` "…OtherSubjectsIsRefused".
  - `TestFollow` "RecordOfAnotherPidStartedBefore…IsNotFollowed".
  - `TestFollow` "ExitedWorkerWithNoRecord…".
- D1 orch strand sets `--no-orch`, including numbered orch and unparsable names: `TestRunAdopt` rows.
- D1 invalid arguments refused before any step: `TestRunWorker` "InvalidRequestIsRefusedBeforeTheLockAndAnyStep".
- D1 worker is detached, registered under `RoleAdopt` and logged (`spawnWorker`): none. `spawnWorker` returns early under `testing.Testing()`, and `TestWorkerArgs` only covers the argument list. Nothing asserts `proc.Detach`, `runproc.Register` or the log call.
- D2 fresh record is written `running` at reconcile before any step: `TestRunWorker` "FreshRunWrites…".
- D2 resume past `owner` starts at deploy: `TestRunWorker` "RecordThroughOwnerResumesAtDeploy…".
- D2 resume at `pairs` or earlier restarts at reconcile: `TestRunWorker` "RecordThroughPairsRunsTheReconcileAgain…" and `TestRecordResumeStep`.
- D2 `--no-orch`, keep-local and drop-local merge rules: `TestRunWorker` "ResumeKeepsTheRecordedNoOrch", "ResumeAddingKeepLocal…" and "ResumeAddingDropLocal…".
- D2 lost lock exits writing nothing: `TestRunWorker` "HeldLockReturnsAtOnceAndLeavesTheRecord".
- D2 worker meeting a running record of the same subjects refuses: `TestRunWorker` "RunningRecordOfThisProcess…".
- D2 `--discard` and its lock: `TestDiscard`, which has rows for removing an interrupted and a stopped record. I saw its names only through row 247; I did not check the held-lock row.
- D2 record read, tolerance, `ReadState` and `SameSubjects`/`Command`: `TestRead`, `TestRecordReadState`, `TestRequestSameSubjects`, `TestRequestCommand`.
- D2/D3 stored inputs read on resume, with the earlier process's parts merged into the end envelope: `TestRunWorker` "DoneEnvelopeMergesThePartsOfStepsFromTwoProcesses".
- D3 step order, a refused step stopping the record, and a held-back project stopping it: `TestAdopt` rows "StepsRunInOrder…", "RefusedOwnerWriteStopsAtTheOwnerStep…" and "MovedTaskWithLocalCommitsStopsNamingItsBeam…".
- Added-after-Discussion, stopped part kept in the envelope:
  - Refused reconcile keeps partial, mutations and the report: `adopt_test.go:413` and `:1009`.
  - Failed deploy names the deploy outcome: "AFailedDeployStopsAtDeploy…".
  - Stored `StoppedPart` is cleared when the step later succeeds: **none**. The only `StoppedPart` mention in tests is `adoptrecord/record_test.go:40`, a round-trip with `{}`.
  - `envelopeOf` merging the stopped part without listing the step as done is covered only indirectly, through the partial and mutations assertions.
- D5 conversion of an owner entry keeping its pause time: `TestAdopt` "OwnerEntryTurnsIntoAnAdoptEntryKeepingItsPauseTime".
- D5 another caller's entry kept, with the slug not paused: "AnotherCallersEntryIsKept…" and "LegacyHoldEntryAndOperatorEntry…".
- D5 re-run skips slugs the record lists as paused: `TestRunWorker` "PausedSlugWhoseEntryTheOperatorLifted…".
- D5 beam pair slug conversion: `adopt_test.go:555` and `:714` (the former names the conversion; row names not fully checked).
- D5 released tasks and front tasks: "ReleasedTaskOfAnAdoptedBeamIsPausedAndAFrontTaskIsNot".
- D5 live pair also paused: "KeepLocalOverALivePairPausesTheRunToo".
- D5 hibernate wake leaves the adopt pause: `hibernateengine` `TestSequence`, the new row.
- D5 `pauserecord.CoverageOf` `Held` for the adopt caller: `TestCoverageOf`, the new rows.
- D5 dobby treats an adopt-caller `Held` run as no pass: only `passes_test.go:49` and `:66` and `owned_test.go:143`, which set `Held` directly. These are existing tests that never see `AdoptCaller` or `HoldCaller`. The mapping from the adopt-caller entry to `Held` is pinned by the `CoverageOf` rows only.
- D5 the dobby's owner lift leaves the adopt pause standing: **none**. `owned_test.go:143` is the nearest test; I did not confirm what it asserts.
- D5 `lym tender pause` answers `already_paused: true` on an adopt-paused slug: **none**. `tendercli/cli_test.go:138` covers only a repeated plain pause.
- D5 run report `paused_by_adopt`, legacy hold, `lym tender resume` and `lym loom resume`: `TestRunReports` rows.
- D6 owner before deploy, with the owner standing and runs paused after a failed deploy: "AFailedDeployStopsAtDeploy…" and "RootDeploysALymOlderThan…AfterTheOwnerWrite".
- D7 `SkipOrch` starts the dobby and no orch or watcher: `TestStart_StartsTheDobbiesFirst`, the new row.
- D7 the beam step passes `NoOrch` to every call: `TestAdopt` "NoOrchPassesNoOrchToEveryBeamStepCall".
- D8 green end only reports, removing the fix task and resetting tries: `TestGreenEnd`. Its rows "ADoneFixTaskAndAnotherProjectsAreLeft" and "AFailedRemovalNamesItsHandRemoval" are pinned. The negative assertions (no lift, no run start) rest on `greenDeps` having no seams for them, so a test cannot fail on them.
- D9 `beam status` adopt section: `TestAdoptSectionOf` for running, interrupted, stopped and done, plus `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams`.
- D9 an undecodable record turning `beam status` into a failure: **none**. The integration test was not checked for that row.
- `holds.json` neither written nor read: no test. This is absence by deletion, and `TestTriesStore` covers only the fix-tries file.
- PATTERN-daemon-wakeups registration: `cmd/lym/daemonwakeups_test.go` lists `follow.go`.

**Findings**

1. MEDIUM, design, `internal/adoptcli/steprun.go:247-257` (`recordStep` clears `StoppedPart`), `:272-275` (`stop` stores it). No test shows that a stored stopped part is dropped when the step later succeeds, or that a resumed envelope omits it. The only `StoppedPart` reference in tests is `internal/adoptrecord/record_test.go:40`. Add a `TestRunWorker` row that stops at a step, resumes and succeeds, then asserts `Record.StoppedPart` is empty and the envelope holds the step's part once.

2. LOW, design, `internal/dobbyengine/passes.go:44-50` (`Held`) and the dobby's owner lift. No dobby test uses `AdoptCaller` or `HoldCaller` to show that the owner lift leaves an adopt pause standing. The existing `Held` rows are at `passes_test.go:49` and `:66` and `owned_test.go:143`, and none were changed in the diff. This is a D5 acceptance criterion. Add an `owned_test.go` row with an `AdoptCaller` entry covering an owned-elsewhere run, asserting the entry stays.

3. LOW, design, `internal/tendercli/cli_test.go:138`. The D5 criterion "`lym tender pause` on an adopt-paused slug answers `already_paused: true`" has no row. The existing test repeats only a plain pause. Add a row that pauses with `AdoptCaller` and then calls `tender pause`.

4. LOW, design, `internal/adoptcli/caller.go:182-228` (`spawnWorker`). No test pins the detach, the registration under `HubOwner` with `RoleAdopt`, or the spawn log. The body returns under `testing.Testing()`. Split the registration into a seam, or add a test at `tmuxkit.Main` level that runs a real spawn.

**Test economy (the two candidates)**

- `TestAdoptFields` (`cli_test.go`): contained, covered by `TestAdopt`. I did not compare assertions (no tool call spent). The parent should check them before filing.
- `TestTriesStore` (`holds_test.go`): contained, covered by `TestFailedEnd`. Same caveat.

**Tool calls**
1. `grep` of test and row names across the diff's test files, to enumerate behaviours (output truncated).
2. `grep` of row names for the remaining files, plus `TestTriesStore`.
3. `grep` for `StoppedPart`, `AdoptCaller` and `already_paused`, plus the diff of the hibernate, pauserecord and orchcli tests. The manifest held no test files.
4. `grep` for `Partial`, dobby `Held` rows, `Register`, `Detach`, `already_paused` and the integration test names, to confirm the gaps above.
