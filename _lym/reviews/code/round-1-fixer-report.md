---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The proposed assertion in the existing two-loops subtest cannot hold. The loser gets the lock once the rival releases it and then refuses with \"an undelivered envelope waits for its waiter\" (gate run, internal/shedverbs/loopwait_test.go:1096), so the lock-loss mark is not written there. The lock grace was a constant of 5 seconds, so no test could reach the branch quickly. I added LoopSpec.LockGrace (zero keeps the constant) and a subtest that holds the lock in-process, sets a 1ms grace and asserts the exact mark and no started child."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Round 1 fixer-report

Commit `ca67146a` holds all six fixes.
The edits of the findings overlap in the same files, so I made one commit instead of one per finding.
A later verify run failed in `internal/slackengine` (`TestRunRelay`) and `internal/shuttleengine/claudeengine`, which this round does not touch, and the gate's triage found no reach into the round's changes.
I changed nothing for them: re-run alone, both packages pass at tier 0 and tier 1.
A later commit breaks four comment lines the line-length gate rejected, in `loop.go`, `loopwait.go` and `loopwait_test.go`, at sentence boundaries.
`lym gate test ./internal/proc ./internal/shedverbs` passes, and `GOOS=windows go vet ./internal/proc` is clean.

## Changes

- **F1:** `proc.BootTime` is replaced by `proc.BootID`, read from `/proc/sys/kernel/random/boot_id` on Linux and always unknown on Windows. `LoopPIDRecord.StartedAt` is replaced by `BootID` (`boot_id`), which `runLoop` fills before its first pid write.
  `LoopSpec.BootTime` is replaced by `LoopSpec.BootID`, and `unmarkedExitCause` names a likely machine stop for a different id and a likely kill for the same id.
  It keeps the old two-cause clause when either id is unknown.
  `loopExitEvidence` takes the boot id string, and `deadLoopStop` passes the record's id while `loopExitedEarlyStop` and the two integration-test sites pass `""`.
  `runLoop` no longer reads `Now` at start, so the two panic tests are back to a clock that panics on its first read.
  The `LoopSpec`, `runLoop`, `LoopPIDRecord` and `internal/proc` docs are updated, and `TestBootTime` is replaced by `TestBootID`.
  The `TestUntilStop_DeadLoopStopsLoopExited` rows are now the different-id, same-id and recorded-id-with-unknown-current rows.
  `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog` asserts the pid record's `BootID` against a stubbed `LoopSpec.BootID`.
- **F2:** `loopExitEvidence` splits out the reconciliation marks (`reconciledStepsMarkText`, now shared with `runLoop`).
  Only a mark outside that set takes the exit branch; a loop with only reconciliation marks gets the not-recorded clause with the cause, then `; loop marks: ...`.
  The function doc and the package doc paragraph say a reconciliation mark is no exit record, and a new row of `TestUntilStop_DeadLoopStopsLoopExited` covers it.
- **F3:** `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog` has a `wantCode` field, 1 for the refusal row and 0 for the failed-envelope-write row, and reports a differing exit code.
- **F4:** see the fixed-differently reason above.
- **F5:** the package doc's pid-file sentence names the process start time and the boot id.
- **F6:** the package doc's early-end sources now include a directory or pid file the loop cannot write.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
