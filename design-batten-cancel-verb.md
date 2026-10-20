# No way to cancel a batten run that stopped before its pair exists

`batten-cancel-verb` · Task · bug, batten

Imported from [issue #775](https://github.com/Knatte18/loomyard/issues/775).

Seen on OSL-1046 (2026-10-20): two batten runs (watchdog-reap-test-xdr, lyxbin-timeout-test-xdr) were started from a prime with uncommitted changes. Worktree-Create went stuck on "source worktree has uncommitted changes", the driver spawned one tackler per run, and both runs ended blocked with no pair created. The operator then cancelled both runs.

There is no verb for that: lyx batten cleanup skips blocked runs, lyx batten pause only adds a pause, and lyx board set-status clears the board entry but leaves the batten records. So the reed overview keeps showing both as blocked, and the only way out is removing _lyx/shed/<slug> and .lyx/shed/<slug> by hand and committing the records side.

Two gaps:
- A cancel (abandon) verb for a batten run, at least one that has not created a pair, which removes its records and scratch and clears the board status.
- Worktree-Create's dirty-source stop is a precondition the operator or orch must fix, so it should halt with that way forward instead of spawning a tackler that cannot settle it.

Workaround: remove the two run directories by literal path and commit the records side with lyx fabric commit, done only while no other run is landing so the prime's records side is not dirty during a Land row.