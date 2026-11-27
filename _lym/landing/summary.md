# Run hubreconcile's Ensure tests in parallel and drop repeated walks

Most of `internal/hubreconcile`'s tier1 time went on tests that ran one at a time.
Each one swapped the package-global detached board sync or the process-global notifier, even when it never asserted on either.
This change installs one recording board sync and one recording notifier for the whole test binary in `TestMain`.
Both key every call by board dir, so each test reads only its own hub's calls and the tests run in parallel.
It changes only test files; no production code changes.

## Timings

Measured on the same machine with `lym gate test --tier 1` (and `--tier 0` where a package has no tier1 test), `-v -count=1`, test binary time:

| Package | Before | After |
|---|---|---|
| `internal/hubreconcile` (tier1) | 19.55 s | 3.65 s |
| `internal/configengine` (no tier1 tests; tier0) | 0.06 s | 0.06 s, unchanged |
| `internal/configsync` (no tier1 tests; tier0) | 2.43 s | 2.43 s, unchanged |

The tier1 call's wall time for `internal/hubreconcile`, build included, went from 29.5 s to 9.6 s.
Net test-line change: 394 added, 450 removed, so 56 fewer lines, all in `internal/hubreconcile`.

## What changed

- `testmain_integration_test.go` holds the two recorders.
  The notifier recorder can answer a board's mail as unposted or failed, so the notifier-outcome rows need no notifier of their own.
- The tests that install the reinterpretation registry are now subtests of one test, `TestEnsure_Reinterpretation`, and run in parallel with each other.
  So are the tests that install the worktree seam, under `TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree`.
  The row that removes the notifier and the row with a refusing seam run on their own before their group's parallel rows start.
- Only the tests that capture the logger still run serially.
- The two unpushed-hub-wide cases become two tests.
  The replayed case needs no log capture, so it runs in parallel.

## Coverage merged or dropped

Every behaviour keeps a test.
- The three stale-build rows become one walk: the board is behind its upstream, and loom's default is deleted beside a current stamp.
  Other tests already walk a board that is up to date.
- `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` is removed.
  `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` walks a board with no remote, and now asserts the same things: one sync asked for, the running key stamped and the board clean.
- The failed-walk row "a failure after the hub-wide files and a template base were written" is removed.
  The reinterpretation subtest that fails the same write now also asserts what that row asserted: the board's status unchanged, no stamp, then a stamp and a clean board after the retry.
- Reinterpretation rows removed: "a rewritten value is mailed", which the rewritten-once subtest covers, and "a hub file with no value to carry", which walks the same unchanged path as the row with a value the rewrite leaves alone.
- Retry walks removed where another row already covers the retry: the EnsureOrWarn broken-config row, the unreadable pair config and the non-mapping template base.
  The arrange helper for an unpushed hub-wide commit no longer runs a second walk in the diverged case.
  The replayed case still checks that the rerun asks for the sync again.
- Tests that need no pair now copy the prime-only hub.

## Design decisions

- Only `internal/hubreconcile` reached 15 s, so it is the only package cut; `internal/configengine` and `internal/configsync` were already far below that and are left as they are.
  Cutting a package that runs in a few seconds saves no test time, so the aim of halving all three packages was narrowed to the packages at 15 s or more.
- No production seam was needed: no test waited out a sleep or a production timeout.
  The one bounded wait, the held-lock row, already passes a 50 ms lock wait.

## Merge danger

**Door:** two-way
**Blast radius:** hubreconcile
