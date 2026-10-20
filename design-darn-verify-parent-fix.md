# Darn-Write verify cannot pass a failure fixed on the parent

`darn-verify-parent-fix` · Note · enhancement, batten

## Seen (2026-10-20, arbiter-verdict-quoting)

Publish failed on `TestOpenWindow_RealTmux`, a bug outside the branch, and batten sent the run back to Darn-Write.
The fix then landed on main through `openwindow-exit-race`, but Darn-Write's verify kept running the failed test on the task branch, which takes main only at Merge-In, so it failed deterministically (verify-7).
The orch had the writer run `lyx fabric merge-in main` by hand.

## Idea

When a verify failure is sent back to Darn-Write and the parent has moved since, merge the parent in before the verify re-runs, or route a failure in a package the branch does not touch to Merge-In instead of Darn-Write.