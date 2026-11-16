# Round 1 fixer-report

## Changes

All five findings were fixed as proposed. Each is its own commit.

- F1: `runplan.PlanEditError` no longer words the mismatch clause. Its tail is the caller's text.
  `websterengine.PlanEditError` now returns "the plan changed since state.json was created; " plus `fingerprintMismatchWayForward`, which matches `runlevel.go:575`.
  The runplan doc comment says the caller's text follows the fingerprints.
- F2: added `recoveredSuspectReasons` in `internal/websterengine/runplan.go`.
  `PersistRecoveryTerminal` calls it, and `export_test.go` aliases `checkRecoveredSuspects` to it, so the tests run the production wiring.
  The self-check showed the existing test never asserted the way-forward text.
  I added `TestCheckRecoveredSuspects_PlanEditNamesRecoverBatchWayForward` (tier 1), which pins the `recover-batch 01` way forward on a plan-edit suspect reason.
- F3: `internal/runplan/doc.go` names the two texts that keep webster's names, the conflict record's begin-batch header and the handle-bind log line.
- F4: `internal/websterengine/doc.go` retitles the section and drops the sentences that repeated runplan's contents.
- F5: the two stale helper names in `beginbatch.go` comments now read `runplan.ChangedPlanFiles` and `runplan.BatchIdentity`.

Verification: `go vet` on both packages is clean. `lym gate test` passes for `./internal/runplan` and `./internal/websterengine` at tier 0, and for `./internal/websterengine` at tier 1 with the new test run.

Gate repair: the new test's doc comment wrapped mid-sentence and ran past 120 characters.
A first repair still split its opening sentence over two lines.
Each sentence of it now sits on one line under 120 characters, and no code changed.
A later verify run failed `TestGofmt_ModuleFilesAreFormatted`: the F1 edit to `internal/runplan/fingerprint.go` had dropped the space after a comma in the `fmt.Errorf` call.
The space is restored, and `gofmt -l` is clean on that file.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me.

## Self-check

- fix-code-correctness, MEDIUM: the F2 test never asserts the way-forward text. Fixed as proposed, with a new tier-1 test.
- fix-proposal, MEDIUM: the same finding. Merged with the one above.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, note, not a finding: the retitled doc section says webster imports no other loop's report schema while it shares runplan's `Report` and `Digest`. That sentence predates this fix, so I did not change it.
