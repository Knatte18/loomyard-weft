---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The review proposed passing the run's discovered-test count into judgeDotnetRedFirst. The count is already in the captured output as the tier summary line, so judgeDotnetRedFirst reads it there through the new gateslot.TierDiscovered and keeps its signature. A run that exits 0 with no test discovered is judged unjudged with noTestRan set, and run ends the call through redFirstCannotJudge naming the filter terms."
  - id: F4
    disposition: fixed-differently
    reason: "The review proposed ending the call as cannot-judge when the output shows no test ran, with no failure report naming a test and no compile error under the test project. The first self-check showed that a run discovering zero tests can also be a timeout or runner crash after tests started, which must keep exit 1. So noTestRan also requires that FindTierTimeout and FindRunnerCrash find nothing in the output."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
---

# Fixer-report, round 1

Every finding is fixed; `lym gate test --tier 0 --tier 1` over `internal/gateslot`, `internal/gitrepo`, `internal/gatecli` and `internal/dotnetimpact` passes.

## What changed

Commits, none pushed:

- `1d205c0f7`, `e4933df95` — F1.
  `WriteTreeAtRevision` admits `filemode.Regular`, `Deprecated` and `Executable` and skips links and submodules.
  The `WriteTreeAtRevision` row of `TestHistoryReads` commits an executable and a symlink in a repository of its own, and asserts the executable's contents and owner execute bit and the link's absence.
- `7ada12dce` — F9.
  A `TestTierFailureNames` row puts a `Ns.C.M:` line under a build header and expects only the tier's failure.
- `67d9de534` — F6.
  `TestGateTest_CheckoutRootMovesTheBuildAndTierDirs` runs `runTest` in process with `checkoutRoot` set, over the stand-in, and asserts that the build and both tiers ran in directories under the checkout.
  It sets the stand-in's role through `t.Setenv`, so it is not parallel, and says so in its comment.
- `543ede42e`, `03c246371` — F2, F3, F4, F5, F7, F8, F10, F11, F12, F13.
  - F2: `gateslot.RedFirstMode` and `Config.RedFirstMode`, a table keyed by scoper name beside `scoperHasGoCoverageMaps`.
    `redfirst.go` branches on the mode, with no scoper literal.
    `TestConfig_RedFirstMode` pins the table.
  - F3, F4: `gateslot.TierDiscovered` sums the discovered counts of the tier summary lines, with `TestTierDiscovered`.
    `judgeDotnetRedFirst` marks an unjudged run `noTestRan` when it discovered no test, or no summary line exists, and no tier timed out or crashed its runner.
    `run` collects those per project and, when nothing is red and no project started tests and judged none, ends with `redFirstCannotJudge`.
    The help text, `doc.go`, `gateslot/redfirst.go`, `gateslot/doc.go`, `docs/overview.md` and `_lym/landing/summary.md` carry the narrowed exit-1 meaning.
    `summary.md` is not tracked by this round's commits.
  - F5, F7, F8: rows of the call test for two test projects, sorted terms with overloads of one method, a concrete test beside an abstract-class one, an export failure and a working-tree deletion of a test-project file; a `TestParseSource_TestMethods` row for a bracket-opening expression body.
  - F10: the keep marker above the judge test.
  - F11: `nearestProject` returns a reason for a directory it cannot read and climbs past one that does not exist, with `TestNearestProject_ClimbsPastAMissingDirectoryAndRefusesAnUnreadableOne`.
  - F12, F13: `doc.go` names both label forms; the overview's unjudged list names the project-cannot-be-told case.

## Gate repair

The comment-format gate flagged eight comment lines this round added: over-long lines in `doc.go`, `redfirstdotnet.go`, `config_test.go`, `gateslot/redfirst.go`, `runlog_test.go` and `blobread.go`, and wrapped breaks in `redfirstdotnet.go` and `test_integration_test.go`.
Each is now split at a sentence boundary, one sentence per line, with no change to its meaning.
The repair is committed on its own.

A later reach-report gate listed the tests that reach each changed function and named none without one, so it needed no new test and the code is unchanged.

## Self-check

Four forks returned reports, saved as `round-1-fixer-report.lens-<lens>.md` beside this file.

- fix-code-correctness, MEDIUM: a timeout or crash with `discovered 0` read as no test ran, so the gate passed it. Fixed: `noTestRan` also needs no timeout or runner-crash line; two judge rows pin it.
- fix-proposal, MEDIUM: the same defect as the line above (duplicate). Fixed with it.
- fix-code-correctness, LOW: the execute-bit assertion depended on the umask. Fixed: it checks the owner's execute bit.
- fix-consistency, LOW: `RedFirstUnjudgedExit`'s doc in `gateslot/redfirst.go` lacked the new causes. Fixed.
- fix-consistency, LOW: `gateslot/doc.go` still said any other code is "a run that judged none of the tests". Fixed.
- fix-affected-set, LOW: `FilesInDirAtRevision` and `FilesUnderDirAtRevision` still filter with `IsRegular()`. Disputed, premise false: their doc comments promise regular files, F1 concerns the tree write alone, and the fix did not change them.
- fix-affected-set, LOW: other `Scoper` literal branches remain in `scope.go`, `tiers.go` and `run.go`. Disputed, premise false: they predate the diff, F2 named only the three branches the diff added, and none is red-first behaviour.
- fix-affected-set, no finding: the `<package>.<Name>` mention in `redfirst.go` is the Go path's own label, and `judgeDotnetRedFirst` has one caller that handles `noTestRan`.

## Disputed

Only fork findings; no review finding is disputed.
Both disputes are in the Self-check section above.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
