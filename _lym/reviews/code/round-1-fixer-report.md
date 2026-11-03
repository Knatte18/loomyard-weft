## Fixer-report, round 1

All ten findings fixed.
Commits: `60501de5` (F1-F10) and the follow-up commit for the self-check findings.
The findings overlap in `prepush.go`, `wiring.go` and `prepush_test.go`, so they landed in one commit rather than one per finding.
`lym gate test` is green at tier 0 over `./internal/loomcli ./internal/fabricengine`, and at tier 1 over `./internal/tendercli ./internal/loomcli` (run before the self-check changes, which touch tier-0 tests only).

## Changes

- **F1** fixed as proposed.
  `prePushCheck` gains a `parentBranch` seam (`readRecordedParentBranch`), and `parentOutcome` picks the code-repository branch of that name from the report.
  An active entry whose parent branch reads `fast_forward` or `diverged` posts one notice; a stopped entry names the parent branch in the stop reason.
  Own change on the second half: the stop reason names the parent only when origin's move stops it (`rewound`, `rewritten`, `diverged`, `gone`), not `live`, `dirty` or `busy`, where origin moved nothing (self-check, fix-proposal).
  Own change on the notice: it is recorded in the run's scratch directory per origin tip (`announcedBefore`), since the run-scope reconcile never moves the parent branch and the state would otherwise repeat on every commit and push (self-check, fix-code-correctness and fix-proposal).
- **F2** fixed as proposed, through extracted seams.
  `loomCommitStatusDeps` builds Commit, Spawn and Push through `guardedCommit`, `guardedSpawn` and `guardedPush`, and `TestPrePushCheckGuardedSeams` drives each with a stopping and a passing stub check.
  The `loomCommitStatusDeps`-over-stubbed-seams test the review named is replaced by that one, because the deps build the check internally and `spawnStatusPush` is a package variable a parallel test cannot swap; the closures' behaviour is the same code.
  `TestCommitRecordsVerb_Success` gains the `push_stopped` row.
- **F3** fixed as proposed: the closure body is `skipsCheck(opts, isPrime, pauseCovers, currentProducer)`, and `TestSkipsCheck` has the rows the review lists, plus unreadable prime and status.
- **F4** fixed as proposed, with `BranchState.stops` exported as `Stops` so loomcli reads the set from its one definition; rows added.
- **F5** fixed as proposed: a failed `landed()` warns and lets the push go; doc comments and an `EndedWithUnreadableLanding` row updated.
- **F6** fixed as proposed, then corrected by self-check.
  `guardedCommit` runs the check before the status commit.
  A stopped commit returns the check's error, which `newCommitStatusSeam` reads as no commit made (no push, no hard error) and `commitRecordsVerb` answers as an ok envelope with `committed` false and `push_stopped`, pushing nothing.
  Tests: `TestNewCommitStatusSeam_StoppedCommitStartsNoPush`, `TestCommitRecordsVerb_StoppedCommitPushesNothing`.
- **F7** fixed as proposed: rows for a failed notice post, several moved own branches, only the stopping own branch named among `level` and `ahead` ones, and an entry of another slug.
- **F8** fixed as proposed, through the extracted seam: `PushTask` is `check.guardedPush(...)`, pinned by the same `guardedPush` test, and `TestPausePair_PausesTheSlugOnceUnderTheDefaultCaller` is added in `internal/tendercli/cli_test.go`.
  No test builds `pullRequestDeps` itself, since it opens no seam without a hub fixture.
- **F9** fixed as proposed in `pattern/PATTERN-owner-machine.md`: the verdict sentences and the reconcile section name the push, and the enforcement bullet and the reconcile section name the commit too.
- **F10**: the Enforcement line is kept as written, since F2, F3 and F8 added tests.
  The line reaches the stops through stub tests of the verdict mapping and the seams.

## Self-check

- fix-code-correctness, MEDIUM: a stopped Commit did not stop `commit-records`' Push, because the pause it wrote turned the second guard into a skip. Fixed with own change (see F6).
- fix-code-correctness, LOW: the parent notice repeats on every guard while the parent stays behind. Fixed with own change (see F1).
- fix-consistency, MEDIUM: same as the first, with `doc.go` and `commitrecords.go` doc claims. Fixed with the F6 change and both docs updated.
- fix-consistency, LOW: `PATTERN-owner-machine.md` bullet named only the push. Fixed as proposed.
- fix-consistency, LOW: the comment above `Commit` in `wiring.go` is silent on a stopped commit. Fixed as proposed: the `loomCommitStatusDeps` and `newCommitStatusSeam` docs now say so.
- fix-proposal, MEDIUM: the parent notice repeats. Merged with the correctness finding; fixed.
- fix-proposal, MEDIUM: a parent in `live`, `dirty` or `busy` is named under "origin moved". Fixed with own change (see F1), with a test row.
- fix-proposal, LOW: `guardedCommit` doc claimed no push after it. Fixed with the F6 change.
- fix-proposal, LOW: F5 lets a push go for an entry origin ended when `landed()` fails. Left as the review chose; see Deferred.
- fix-affected-set, MEDIUM and LOW: the same stopped-commit finding and `drivewiring.go`'s `commitRecordsVerb` entry point. Fixed in `commitRecordsVerb`, which covers it.
- No fork returned no report.

## Disputed

None.

## Deferred

- F5's trade-off, raised by fix-proposal: a transient `landed()` failure lets the push go for an entry origin ended, which can push a branch origin removed, where the previous behaviour paused a healthy run on the same failure.
  The review chose the push-goes side to match every other failed read of the check; reversing it is an operator decision on that trade-off.
