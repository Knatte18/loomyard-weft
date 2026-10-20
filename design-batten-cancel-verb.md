# No way to cancel a batten run that stopped before its pair exists

`batten-cancel-verb` · Task · bug, batten

Depends on: [`run-process-ownership`](design-run-process-ownership.md), [`darn-verify-fixes`](design-darn-verify-fixes.md)

Imported from [issue #775](https://github.com/Knatte18/loomyard/issues/775).

Seen on OSL-1046 (2026-10-20): two batten runs (watchdog-reap-test-xdr, lyxbin-timeout-test-xdr) were started from a prime with uncommitted changes. Worktree-Create went stuck on "source worktree has uncommitted changes", the driver spawned one tackler per run, and both runs ended blocked with no pair created. The operator then cancelled both runs.

There is no verb for that: lyx batten cleanup skips blocked runs, lyx batten pause only adds a pause, and lyx board set-status clears the board entry but leaves the batten records. So the reed overview keeps showing both as blocked, and the only way out is removing _lyx/shed/<slug> and .lyx/shed/<slug> by hand and committing the records side.

Two gaps:
- A cancel (abandon) verb for a batten run, at least one that has not created a pair, which removes its records and scratch and clears the board status.
- Worktree-Create's dirty-source stop is a precondition the operator or orch must fix, so it should halt with that way forward instead of spawning a tackler that cannot settle it.

## Operator decision (2026-10-20): batten refuses a dirty source up front

`lyx batten run` checks that the source worktree is clean before it does anything, and refuses with the dirty paths and the way forward when it is not: no run record, no pair and no tackler.
The orch should not have to run `git status` before every start; batten owns that precondition.
A dirty source never spawns a tackler, at start or at Worktree-Create.

Workaround: remove the two run directories by literal path and commit the records side with lyx fabric commit, done only while no other run is landing so the prime's records side is not dirty during a Land row.