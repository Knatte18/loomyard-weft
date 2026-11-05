# Drift rework: all batches done

Every batch in the card list reached `done`.
This session ran the last six batches; the earlier eight were already recorded when it started.

## Earlier batches (from the progress trail)

- 01 fabric-trailer-commits, 05 drift-expectation-parser, 09 drift-current-capture, 11 webster-runlevel-entry-report, 13 drift-expect-rows, 15 drift-judge-seat, 16 drift-verify-lines and 17 drift-verify-command were done.
- Pattern departures reported for batch 05 and batch 15, both under `PATTERN-fabric-vocabulary`: side-named identifiers were renamed (`TrailerCommit.WarpSHA` to `CodeHead`; `Deps.WeftChangedSince`, `Deps.LoomWeftDirs` and `fabricengine.WeftPathsChangedSince` to `RecordsChangedSince`, `LoomRecordsDirs` and `RecordsPathsChangedSince`).

## This session

- 20 drift-verdict (head `5f9bfd69`): the verdict renders from the committed drift records, and the drift paragraph is in the darn writer's, Merriam's and the card fork's stencils. Deviations: `internal/driftengine/verdict.go`, `internal/driftengine/verdict_test.go`, `internal/loomcli/driftwiring.go`, `internal/loomcli/wiring_publish_integration_test.go`.
- 22 driftcli-retire-verbs (head `46f2cc53`): the `scope`, `accept`, `withdraw`, `waive` and `compare` verbs, `capture --side` and the DriftRecheck engine are removed. Deviations: `internal/driftcli/capture_integration_test.go`, `internal/shedrecipe/coverage_guard_test.go`.
- 24 drift-retire-old-seats (head `0dd6cb13`): the baseline and verify agents' seats, stencils and seams are removed. Deviation: `internal/driftengine/seats.go`.
- 25 driftengine-retire-acceptance (head `d115996b`): acceptances, the run budget and side capture are removed. Deviations: `internal/driftengine/capture.go`, `records.go`, `steprun.go`, `verbs.go` and `verbs_test.go`.
- 26 drift-retire-skip-keys (head `3f486cda`): the `skip` and `verify_max_runs` keys and the `{skip_file}` placeholder are retired. Deviations: `internal/driftcli/wiring.go`, `internal/driftengine/steprun.go`, `internal/loomcli/driftwiring.go`.
- 27 drift-retire-verify-gate-key (head `6248ad8b`): the `drift-verify` gate key is removed. No deviations.

No batch in this session reported pattern departures or renames, and none needed recovery.
The scope guard's result is the deviation list under each batch; all of those files are informational.

## Verify gate findings

The first verify attempt failed on these identities from the findings file:

- `github.com/Knatte18/loomyard/cmd/lym` (package).
- `github.com/Knatte18/loomyard/internal/hubreconcile.TestEnsure_ReinterpretationMarkerIsCommittedAndMailsOnlyAFlaggedValue/root-anchored_hub_with_a_dot_dir` (test).
- `github.com/Knatte18/loomyard/internal/hubreconcile.TestEnsure_ReinterpretationMarkerIsCommittedAndMailsOnlyAFlaggedValue/anchored_hub_with_no_dir_set` (test).

The findings file noted that no recorded card touched a failing package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `34fa6087a017058b043f09f57abb396c41ecd10a` fix: drift amend and defer Use lines follow the CLI form, and the reinterpretation fixtures set diff so only the dir flag mails


## Plan rebaselined

webster: the plan changed since the run recorded it and was rebaselined on entry, keeping 8 begun batch(es) and accepting 22-driftcli-retire-verbs.md, 25-driftengine-retire-acceptance.md
