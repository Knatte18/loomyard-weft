# TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand flakes on records index.lock and halts Merge-In

`smokestart-index-lock-flake` · Task · bug, loom, fabric · running · Code-Burler-Fix

Imported from [issue #856](https://github.com/Knatte18/loomyard/issues/856).

The tier-2 Merge-In verify failed once on `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` (`internal/loomcli/smoke_starttail_test.go`).
The second `lyx loom start` got `index.lock: File exists` on the pair's weft worktree, so a process the first start left behind still held the records index.
The branch under verify changed nothing in `internal/loomcli` or the records git path.
The test passed 5 of 5 times alone, the full tier 2 passed on the same HEAD, and no other gate log on record shows the failure.

Why it is abnormal: an unrelated flake escalated the run to a human.
The loom halted `blocked` at Merge-In with anomaly `escalation-to-human`: "landing gate failed after merging parent branch "main" (exit code 1)", way forward `lyx loom goto --to Darn-Write` then `lyx loom resume`.

Cause: the test's repeated starts do not wait for the records index lock the way `TestSmokeBootstrapLifecycle` does with `waitRecordsIndexLockGone`.
It will recur under load.

What to do: make the test wait with `waitRecordsIndexLockGone` between starts, or make `lyx loom start` tolerate a concurrent records commit.

Provenance: task `verify-fix-escalation`; notes `Darn-Write.md`, `loom-halt.md`.
