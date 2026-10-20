# reedcli TestWatchdogReap/NeverEnteredOrphanIsStillReaped times out under Cortex XDR (1.8 s window)

`watchdog-reap-test-xdr` · Task · bug, reed · running · Publish

Imported from [issue #769](https://github.com/Knatte18/loomyard/issues/769).

Seen on OSL-1046 (Ubuntu 24.04, Cortex XDR), run gate-fewer-new-binaries, Publish verify (verify-7.log, 2026-10-20T10:14Z):

--- FAIL: TestWatchdogReap/NeverEnteredOrphanIsStillReaped (11.47s)
    watchdogreap_integration_test.go:194: condition not met within 1.8s

The subtest waits DiscoveryCycle*OrphanGoneCycles*20 from compressedReapTiming(), 1.8 s, for the reap loop to remove the orphaned tmux session.
The subtest itself ran 11.47 s, so the setup (fixture, tmux, worktree removal) was slow; Cortex XDR holds new process images, so the reap loop's tmux and git calls can miss a window that tight.
The failure is unrelated to the branch (gatecli, loomshed), and it sent the run back to Darn-Write for another full round.

Expected: the wait is derived from a bound that tolerates a slow host (a fixed floor of several seconds, or a scaled timing), as with #767.