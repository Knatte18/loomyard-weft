# Fixer-report, code review round 1

All twelve findings are fixed.
`lym gate test` over `internal/testcover`, `internal/gatecli` and `internal/gateslot`, tiers 0 and 1, passed after the changes.
`TestGateRedundant` passed again after the self-check edits.
The commits group findings by file, since several findings share one file: F1, F2, F3, F4, F5, F11 and F12 touch the same gatecli files.

## Changes

- F1, F5: `mutationCheck.run` returns each contained test's `CoveringTests` pick, and `runRedundant` prints it for candidates and refuted tests.
  `TestGateRedundant` no longer zeroes `Covering` and asserts each list exactly.
  The fixture's `TestSourceScan` sorts before the Tally tests and overlaps them fully, so a return to the coverage pass's list names it whenever the greedy pass keeps it.
- F2: the cover pass of `runMutants` queues a covering test that ran the mutant as a candidate and killed it, and clears that kill once every job is chosen, so the alone rerun decides it.
  The fixture reaches the branch: `TestAll` and `TestDoubleChecked` cover each other.
- F3: `cov.go` gains `Timeout`, whose constant does not compile behind the generic helper.
  The test asserts `Unbuildable == 1` and `Mutants == 12`, with the verdicts unchanged.
- F4: `TestGuarded` carries `//lym:guard` and is asserted in `NotJudged` with `ReasonSourceCheck`.
- F6: `//testtiming:keep` above `TestPlanMutants`.
- F7: `ScanSourceReaders` keeps the call path apart from the memo, and records a false answer only when its walk met no function above it.
  The test calls the scan 16 times, since the map's iteration order decides which cycle member is walked first.
- F8: a selector call on an imported package skips the method lookup.
  `TestImportedFunction` and a `scanner.Run` method pin it.
- F9: `template.yaml` and the gateslot `doc.go` name the mutation check among the runs each timeout key bounds and say a timed-out mutant run counts as a kill.
- F10: README line 78 says the verb confirms by mutation.
- F11: `Tally`'s dropped `n -= 1` makes an endless loop.
  The fixture sets `redundancy_test_timeout_sec: 10`, and `MutantsKilled: 7` counts that mutant as killed.
- F12: a multi-line mutant row in `TestMutantPlan_UnbuildableMutants`, and a stale-map row with a file on each side only (`gone.go`, `lacking.go`).
  The covering-kill-rerun part is reached by F2's branch in the fixture.
  The candidate failing its unmutated run was dropped, under the finding's own clause: the map run records a test that fails alone as failed, so `TestFails` reaches `not_judged` before `runMutants`.
- Self-check edits: `gatecli/doc.go` states that the rerun bound concerns a candidate's own kill; the fixture's test bound went from 4 to 10 seconds.

- Gate findings: five comment lines I added wrapped mid-sentence or ran to 120 characters.
  They now break one sentence per line, in `gatecli/doc.go`, `redundant_integration_test.go` and `redundantmutants.go`.
  No code changed.

## Self-check

- fix-consistency, LOW, the `doc.go` rerun bound read as contradicting the F2 text: fixed as proposed.
- fix-code-correctness, LOW, no deterministic test fails with F2 reverted: disputed, every fix worse.
- fix-code-correctness, LOW, and fix-proposal, MEDIUM, the 4 second bound can time out a healthy run: fixed with an own change, a 10 second bound instead of the 10 to 20 the fork named; the endless mutant still ends each run in 10 seconds.
- fix-proposal, LOW, F5's assertion does not fail on every run with the old list restored: disputed, every fix worse.
- fix-proposal, LOW, F8's import check misses a local that shadows an import name: disputed, every fix worse.
- fix-proposal, LOW, F7's unmemoized answers are re-walked: no change, the fork itself found no finding at the sizes of real test files.
- fix-affected-set: no findings.

## Disputed

- fix-code-correctness, F2 test, every fix worse: pinning the alone rerun needs a fake of `runJobs`, a method of the package's own `mutationCheck`.
  The testing rules forbid faking one's own code, and a rerun of an undisturbed test returns the same kill, so no public-surface assertion differs.
- fix-proposal, F5, every fix worse: the coverage pass's list names `TestSourceScan` only when its greedy pass keeps it, which depends on the run's seconds.
  Making that certain needs `TestSourceScan` to hold a block no other test covers, which makes it non-contained and removes the overlap the fixture relies on.
- fix-proposal, F8, every fix worse: telling a shadowing local from an import needs type information, and `loadPkgSource` parses only.
  The reviewer's own model, `spawnscan.analyzeCall`, matches by name the same way.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.
