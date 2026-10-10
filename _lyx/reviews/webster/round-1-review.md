---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/websterengine/runlevel.go:480
    summary: the step-keyed zero-batch refusal and quarry transient Run raises after an accepted auto-rebaseline are reworded refusals no test reaches
  - id: F2
    severity: LOW
    class: design
    location: internal/websterengine/begun_test.go:44
    summary: TestEditedDoneCards repeats the begun-but-not-done assertion TestNewPlanGate already pins, and the gate row meant for an unbegun card actually builds an in-flight batch
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/wait.go:746
    summary: expiredTurnEnd returns early on an empty waiting list, so an ungated EventWaiting with no outstanding task and every output file present no longer counts at once
  - id: F4
    severity: NIT
    class: consistency
    location: internal/shuttleengine/waitshadow.go:77
    summary: loopAgreesWithState still says the loop has waited out tasks, though no shell or fork is waited out any more
---

## Range and method

No `_lyx/loom/rework/` directory exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, the 27 commits from `1: gitexec-run-stdin` through the `fix:` commit after card 26 (`status.json` gives `product.parent` = `main`).
I read every card in `_lyx/plan/`, the diff for the cards the focus names, and ran `lyx gate test` over the touched packages in the unit tier and in the `integration` tier; both exited 0.
The `llm` and `tmux` tiers were not run.

### [BLOCKING] Reworded post-rebaseline refusals have no test

**Location:** `internal/websterengine/runlevel.go:474-482` (`recordedBatches`, the zero-batch branch) and `internal/websterengine/runlevel.go:610-618` (the quarry transient).

**Issue:** Card 4 widens three `Run` refusals once an auto-rebaseline was accepted: the plan-validation refusal, the zero-batch refusal and the quarry transient.
Each then names `rebaselineStep` and wraps `ErrAutoRebaseline`, which is what turns a hard row failure into `Stuck` in `shedadapters.WebsterProducer.Call`.
`contracts/specs/refusal-spec.md` rows `Webster row: auto-rebaseline refused` and `zero execution batches` name those variants.
`TestRun_AutoRebaseline` reaches only the plan-validation variant (`a blocking plan-validation finding after the rebaseline blocks the row`).
`grep -rn "zero execution batches\|produced zero\|zeroBatchesError" internal --include=*_test.go` finds nothing, and the only quarry-unavailable test (`runlevel_test.go:632`) runs without a rebaseline, so the `rebaselineStep != ""` arm of the quarry text and the step-keyed `zeroBatchesError` text are never executed.
`PATTERN-refusal-way-forward` requires that a test reaches a refusal added or reworded in webster, and the rubric lists a behavior the diff adds with no coverage as a finding.

**Fix:** Add two rows to `TestRun_AutoRebaseline`'s refusal table, reusing its `autoRun` fixture.
One row makes the restamped plan form no batch and asserts `ErrAutoRebaseline`, the re-step text, and neither `lyx webster run` nor `lyx webster rebaseline`.
The other makes validation return `planindex.ErrQuarryUnavailable` after an accepted rebaseline and asserts the same.

### [LOW] `TestEditedDoneCards` repeats a gate row

**Location:** `internal/websterengine/begun_test.go:44`, against `internal/loomshed/gates_test.go` (`EditedUnbegunCardReportsNothing`).

**Issue:** `TestEditedDoneCards` asserts that an edited card of a begun but not-done batch reports nothing.
`TestNewPlanGate`'s row `EditedUnbegunCardReportsNothing` builds `batchOne(false, "", hashes)`, a non-terminal batch that has begun, edits the card and asserts no `done-card-edited` finding, so it pins the same behavior through `ValidatePlan` and `EditedDoneCards`.
The row is also misnamed: a card in no batch at all is not exercised anywhere.
The other two assertions of `TestEditedDoneCards` (a recorded id the plan lacks, and a nil state) are not covered elsewhere and stay.

**Fix:** Drop the begun-batch fixture and assertion from `TestEditedDoneCards`, or rename the gate row to what it builds, `EditedInFlightCardReportsNothing`, so the unbegun case is not claimed as pinned.

### [LOW] Empty waiting list changes the ungated at-once rule

**Location:** `internal/shuttleengine/wait.go:746-749` (`expiredTurnEnd`).

**Issue:** Card 22 says an ungated run keeps today's rule: a turn end with every output file present counts at once, whatever is outstanding.
Before the diff, an ungated `EventWaiting` with every output file present returned `OutcomeDone` from `pollEventsTick` before `expiredTurnEnd` was consulted.
Now every `EventWaiting` goes through `expiredTurnEnd`, whose first line returns `"", nil, nil` for an empty `waitingTasks`, so an ungated `EventWaiting` with an empty list and the files present waits for the deadline or the next event.
`claudeengine` emits `EventWaiting` only for a non-empty list, so no provider reaches this today, but the test engine (`waitingEngine{}` in `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival`) does emit one, and the empty-list guard exists only for the gated case the card names.

**Fix:** Apply the empty-list hold only when `len(run.gate) > 0`, or have `pollEventsTick` return `OutcomeDone` for an ungated `EventWaiting` with the files present before calling `expiredTurnEnd`.
A row in `TestPollEventsTick_OutstandingBackgroundWork` with an empty task list pins it.

### [NIT] Stale "waited out" comment

**Location:** `internal/shuttleengine/waitshadow.go:77`.

**Issue:** The comment on `loopAgreesWithState` explains the `busyOnBackground` allowance for a held or done run as "the loop has waited out or finished past tasks the turn end still reports".
After card 22 no shell or fork is waited out, and `expiredTurnEnd` never returns a held turn end, so the first half of the reason no longer describes the code.

**Fix:** Reword the reason to what still produces the pairing, or drop the clause if the `loopHeld` allowance is no longer reachable.

## Focus verdicts

- **Cards 14, 15, 16, injection.** Holds. No production file in `internal/websterengine` imports `fabricengine` (only `_test.go` files do).
  `ClassifyViolation`, `RecordDeps`, `RunDeps` and `RecoverDeps` take `ReadOnly func(cmd string) bool`; `webstercli`'s `runDeps`, `recordBatchCmd` and `recoverBatchCmd`, `acceptBatch`, and `loomCLI.wire` supply `fabricengine.IsReadOnlyCommand`; a nil classifier accepts nothing, and `AcceptBatchFabricReference` is reached from `RecoverSpawnOrAttach` only through `allReadOnlyFabricReferences`, which is false for nil.
  I read the allow-list (`plainWord`, `gitForm.admits`, `readOnlyLyxSegment`, `readOnlyGitSegment`) against the card's lists and found no form admitted beyond them; the `||` replacement keeps rune alignment.
- **Cards 3, 4, 5.** Holds. Every `Rebaseline` clause that named a manual verb is keyed by `deps.Step` (`followUpLanding`, `transientWayForward`, `freshRestartSteps(stepOrRun(...))`, `CheckBatchOrder(batches, step)` on both `rebaselineBatches` branches), and the auto path names every changed card file, the same input an operator naming each with `--card` gives `Rebaseline`.
  The remaining mentions of `--card` in a step-set text are descriptions inside the batch finding (`--card cannot accept the edit`), not instructions.
  The one gap is F1.
- **Cards 24, 25.** Holds. `RecoverSpawnOrAttach` is the only caller of `recoverSpawn`, so the cap of two at `prior.Recoveries >= maxRecoveries` (waived only by `rendersNewAmendment`) covers every spawn; `BeginBatch` carries both fields over a re-begin and `reset --to batch-start` clears them.
  `RecoveryRetry` is read by `recoverBatchCmd`, `statusCmd` and `BeginBatch`, so resume sees the same verdict from the record and git alone; it is true only for a terminal dead recovery below the cap with a non-merge first-parent commit past a non-empty `RecoveryStartSHA`, and the engine does not forbid an operator-run second recovery, as the card's bound states.
- **Cards 18, 20, 17.** Holds. `readerVerb` admits exactly `FabricReaderVerbs` with `shortname` argument-free and `prune`/`cleanup` without `--apply`; `TestSandboxedRoleGuard` walks every registered subcommand and fails on an unpinned new one, and `--help` never reaches the pre-run.
  Every `lyx fabric` way forward a refused session reaches carries the orch clause (grepped across non-fabric production code).
  `gatecli` and `verifytree` drop the strand name on every branch.
- **Cards 6, 7.** Holds. `EditedDoneCards` compares each done batch's card bytes with the `CardHashes` recorded at begin, so only done cards edited since their batch began are flagged; `ValidatePlan` appends one blocking `done-card-edited` per id and returns the read error for an unreadable card file.
  F2 is the only note.
- **Cards 10, 11, 12.** Holds. `splitChain` is used by both `moduleWideCommands` and `crossModuleCommands`, treats `||` as one separator, ignores separators inside single- or double-quoted text and joins continuations; the `cd` rule stops the scan after `|` and `||`.
  `checkPatternEntryLineCap` scans fenced spans only, counts runes and runs for every entry point `validate` serves.
  The spec rows and `doc.go` passages for the new checks and the `card not done` wording landed in the same commits.
- **Cards 21, 22, 23.** Holds apart from F3 and F4. `TurnEndedAfter` counts a turn end only with nothing outstanding; `pollEventsTick`/`expiredTurnEnd` apply one rule to both shell signals, `Result.EndedShells` is filled in `finalize` from `waitingTasks` or `countedTasks` on every exit, and `anchoredRunPattern`/`publishFailureCommand` emit one top-level `-run` step per package in first-appearance order.
- **Test economy and budget.** The `test-budget.yaml` rows for `fabriccli`, `loomshed`, `planparser`, `verifytree` and `websterengine` rise by exactly the new top-level tests in each; the `shuttleengine` row is unchanged although the package lost net two top-level tests.
  Redundancy and coverage findings are F1 and F2.
