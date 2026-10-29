# Fixer-report, code round 1

## Changes

All three findings fixed as proposed, one commit each; `lym gate test` on `internal/gateslot`, `internal/gatecli` and `internal/adoptcli` is green.

- F1 (MEDIUM), `internal/gateslot/runlog_test.go`: the round-trip record now carries `Failures`, read back equal; the omitted-key check asserts a `RunRecord{}` encodes without `failures` and a record lacking the key reads with none.
  The optional wiring assertion for `internal/gatecli/tierrun.go:115` is not added: no test in `internal/gatecli` calls `runTier`, so the review's condition (an existing fixture) does not hold.
- F2 (LOW), `internal/gatecli/counters_test.go`: a go-json row with `TestA` and `TestAB` both failing and no subtest, both wanted in `wantFailures`.
- F3 (NIT), `internal/adoptcli/check_test.go`: a `TestRunCheck` case with `failedTier(0)` and no failures, asserting the notice is the summary line then `log ` plus the log path.

## Self-check

- fix-consistency, LOW: subtest name `RecordWithoutLaunchesReadsWithNone` omitted failures. Fixed as proposed: renamed `RecordWithoutLaunchesOrFailuresReadsWithNone`.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
