---
verdict: APPROVED
prior:
  - id: F1
    status: resolved
    answer: boardSyncRecorder.record now counts the call under the lock and returns boardengine.SpawnSync(boardPath), the function walk.go's spawnBoardSync holds in production, so it is a counting wrapper the stand-ins rule allows.
  - id: F2
    status: resolved
    answer: The broken-config row of TestEnsureOrWarn_WarnsWithTheWayForward again repairs the file, calls EnsureOrWarn a second time and asserts no way-forward warning and a written stamp; its body matches the base row's.
  - id: F3
    status: open
  - id: F4
    status: resolved
    answer: The summary now reads 390 added and 428 removed, which matches git diff --stat 2326ef4da..HEAD (390 insertions, 428 deletions).
  - id: F5
    status: resolved
    answer: Commit 6bbb44653 carries the correction to f2ebdb2c1's body, naming the refusing-seam row as running alone.
findings:
  - id: F1
    carries: F3
    severity: MEDIUM
    class: decision
    location: internal/hubreconcile/testmain_integration_test.go:402
    summary: The notifier recorder scripts the posted, unposted and error answers of the notifier orchcli installs in production. The base already held this standing fake, but the diff installs it for every test of the binary, so walks that ran with no notifier now mail into a scripted one.
    fix: Leave the code in place and defer this to the run's parent. The parent rules whether the standing notifier fake may serve every test of the binary until a sweep replaces it with the production notifier. A fixer cannot drive the real notifier inside this test-only cut, and the decision record holds no added entry that settles it.
---

### [MEDIUM] The binary-wide notifier recorder is a standing fake of orchcli's notifier, now serving every test

**Location:** `internal/hubreconcile/testmain_integration_test.go:402`

**Issue:** Carried from round 1's F3, which the parent holds.
`mailRecorder.send` answers each mail with a hand-set posted flag and error, where production fills `hubreconcile.InstallNotifier` from `internal/orchcli/reconcilenotifier.go`.
The base held the same fake per test (`recordNoticesAnswering`); the diff installs it once in `TestMain`, so it now answers every walk of the binary.
No added entry in the decision record settles it, so it stays open for the parent's ruling.

## Focus verdicts

- **TestMain's seams.**
  The board-sync recorder passes each call to `boardengine.SpawnSync`, the value `spawnBoardSync` holds in production (`internal/hubreconcile/walk.go:218`); it meets PATTERN-test-stand-ins.
  It is installed before any test runs, so the unsynchronised package variable is never swapped under a running test.
  The notifier recorder is the carried F3 above.
  Ordering holds: `TestEnsure_Reinterpretation` and `TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree` do not call `t.Parallel`, so their parallel subtests run only beside each other, and the registry and worktree seam they install never reach another test's walk.
- **Removed and widened assertions in `ensure_integration_test.go`.**
  The EnsureOrWarn after-repair walk is back, identical to the base.
  The retry walks dropped from the "unreadable pair config" and non-mapping template base rows are held, no looser, by the retries of the "unparseable pair config" and "unparseable hub-wide config" rows.
  The "failure after the hub-wide write" row's assertions moved into `runFailedWalkAfterAReinterpretation`, which also asserts the board's status, the absent stamp and the clean retry.
  `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp`'s assertions moved into `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`.
- **Summary and commit claims.**
  The net test-line figure matches the range's diff stat.
  The corrected count of removed retry walks matches the diff.

## Rejected

These items were weighed and not raised.
This round is incremental, so a new finding must be BLOCKING and sit on a changed hunk, and none of these is BLOCKING.

- Keying the recorders by board dir widened some assertions:
  `slices.Equal(*spawned, []string{board})` and `len(*spawned) != 0` became `boardSyncs.count(board)`, and the no-mail rows read only `orchMail.lines(board)`.
  A sync or mail aimed at a wrong path in the same hub now goes unnoticed.
  No commit carries a `Loosened-Check:` trailer for this.
  It was present at round 1's review, and its risk is low.
- The summary's after-figures (3.65 s, 9.6 s wall) predate HEAD, as the summary itself says.
  My tier1 run at HEAD gave 18.15 s test-binary time and 28.1 s wall, under a load average of 38 to 51 with other gates' tier1 and tier2 runs going at the same time.
  That run cannot confirm or refute the figures, and `_lym/landing/summary.md` is not a changed file this round.
- The summary's "Only the tests that capture the logger still run serially" contradicts its previous bullet:
  `TestEnsure_Reinterpretation` and the heddle-render test also run as serial top-level tests, along with their notifier-removing and refusing-seam rows.
  It is LOW, and the summary is not a changed file this round.
