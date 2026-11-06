# Round 1 fixer report

All six findings fixed as proposed. Tier0 gate green on `lensreport`, `seatengine`, `burlerengine` and `shedadapters`.

## Changes

- **F1** — `internal/burlerengine/assist_test.go`: `TestEngine_Review_AssistantPromptFollowsItsFan` lost its `wantInText` and `wantAbsent` prose strings; the rows now pin the stencil through the lens report paths Go fills (present for the fanned row, absent for the solo row). `internal/seatengine/prompt_test.go`: the fanned subtest lost its prose and absence checks and keeps the `output_files` list assertion.
- **F2** — `internal/shedadapters/burlerreview.go`: `acceptedExit` clears `AssistEnding` and `AssistLensesMissing` before setting them from `result.Assists`. `TestBurlerReviewProducer_Call_RecordsAssistOutcome` gained a row that pre-seeds a stale ending and missing lens, then accepts a round with no assistant. `recordRoundAttempt` was not changed, so its doc stays true.
- **F3** — `TestEngine_Review_AssistantPromptFollowsItsFan` keeps the `ReviewResult` and asserts `Assists[0].MissingLenses` equals the fan's lens names in fan order for the fanned row and is empty for the solo row.
- **F4** — `internal/lensreport/lensreport.go`: `Returned` reads only the opening fenced block's `outcome` line, trimmed, with the standard library alone. `TestReturned` gained CRLF, trailing-whitespace, no-block-with-the-line and line-after-the-block rows.
- **F5** — comments reworded in `internal/seatengine/table.go`, `internal/seatengine/doc.go` and `internal/seatengine/engine.go`.
- **F6** — the `seatSpec` `OutputFiles` assertion deleted from the seatengine fanned subtest (done together with F1).

Commits: one per finding group (F2, F4, F5, and F1 with F3 and F6, which share two test files), then one for the fork findings below.

## Self-check

- fix-code-correctness, LOW: `Returned` held an unreachable `len(lines) == 0` guard — fixed as proposed.
- fix-consistency, LOW: the fanned subtest's name no longer matched its one assertion — fixed as proposed, renamed to `AFannedAssistantListsItsSubagentReportsAfterItsOutputs`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

All four forks returned a report; their files sit beside this one as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.
