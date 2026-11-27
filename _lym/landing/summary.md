# Run hubreconcile's Ensure tests in parallel and drop repeated walks

The aim is to cut the tier1 test time of `internal/hubreconcile` at least in half.
Many of its tests ran one at a time because they swapped the package-global detached board sync
or the process-global notifier.
`TestMain` now installs one recording board sync and one recording notifier for the whole test binary.
Both key every call by board dir, so each test reads only its own hub's calls and can run in parallel.
The board sync recorder counts the call and passes it on to `boardengine.SpawnSync`.
Only test files change.

## Timings and size

Measured on one machine with `lym gate test --tier 1` (`--tier 0` where a package has no tier1 test),
`-v -count=1`, as test binary time:

| Package | Before | After |
|---|---|---|
| `internal/hubreconcile` (tier1) | 19.55 s | 3.65 s |
| `internal/configengine` (no tier1 tests; tier0) | 0.06 s | 0.06 s, unchanged |
| `internal/configsync` (no tier1 tests; tier0) | 2.43 s | 2.43 s, unchanged |

The wall time of the `internal/hubreconcile` tier1 call, build included, went from 29.5 s to 9.6 s.

The `internal/hubreconcile` after figures are older than the final code:
they were measured before the broken-config row of `TestEnsureOrWarn_WarnsWithTheWayForward` got its second,
serial walk back.
A later run of the final code on a heavily loaded machine (load average 38 to 51) took 18.15 s of test binary time
and 28.1 s of wall time.
That run neither confirms nor refutes the halving.

Net test-line change: 390 added, 428 removed, so 38 fewer lines, all in `internal/hubreconcile`.

## What changed

- `testmain_integration_test.go` holds the two recorders.
  The notifier recorder can answer a board's mail as unposted or failed.
- The tests that install the reinterpretation registry are now subtests of `TestEnsure_Reinterpretation`.
  The tests that install the worktree seam are subtests of
  `TestEnsure_RendersTheHeddleFileTargetsOfEveryWalkedWorktree`.
  Within each, the subtests run in parallel,
  except the one that removes the notifier and the one with a refusing seam, which run alone first.
- Those two parent tests and the tests that capture the logger still run serially against the rest of the package.
- The two unpushed-hub-wide-commit cases are now two tests, and the replayed case runs in parallel.
- Tests that need no pair copy the prime-only hub.

## Coverage merged or dropped

- The three stale-build rows are one walk: the board is behind its upstream,
  and loom's default is deleted beside a current stamp.
- `TestEnsure_BoardWithNoRemoteCommitsHubWideConfigAndWritesStamp` is removed.
  `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` walks a board with no remote.
  It already asserted a clean board, and now also asserts one board sync and the running key stamped.
  The removed test's assertion that the walk adds exactly one local board commit is dropped.
- The failed-walk row for a failure after the hub-wide files and a template base were written is removed.
  The reinterpretation subtest that fails the same write takes over its assertions,
  except that the error message names the board directory; it checks the error's `Worktree` field instead.
- Two reinterpretation rows are removed: a rewritten value being mailed, which the rewritten-once subtest covers,
  and a hub file with no value to carry, which takes the same path as a value the rewrite leaves alone.
- The retry walks of the unreadable-pair-config and non-mapping-template-base rows are removed;
  other rows of that test still retry.
- The diverged unpushed-commit case no longer reruns the walk before the upstream moves.
  The replayed case still checks that the rerun asks for the sync again.

## Design decisions

- Only packages that took 15 s or more are cut.
  `internal/hubreconcile` was the only one;
  cutting a package that runs in a few seconds saves no meaningful test time,
  so `internal/configengine` and `internal/configsync` are left as they are.
- No production seam was needed.
  The held-lock cases already pass a 50 ms lock wait.
- Keying the recorders by board dir loosens some assertions:
  a board sync or a mail aimed at a wrong path would not be noticed by the test of the right board.

## To check by hand

- Re-time `internal/hubreconcile` tier1 on an idle machine with the command above,
  and confirm that it is at most half of the before figure.
- The notifier recorder in `internal/hubreconcile/testmain_integration_test.go` (`mailRecorder.send`)
  is a standing fake of the notifier that `internal/orchcli` installs in production,
  with hand-set posted, unposted and error answers.
  A per-test fake of this kind existed before; it is now installed for the whole test binary,
  so walks that ran with no notifier now mail into it.
  Decide whether that is acceptable as a test stand-in.

## Merge danger

**Door:** two-way
**Blast radius:** hubreconcile
