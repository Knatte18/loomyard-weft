# Merge-In verify failed on a loomcli smoke flake the branch does not touch

The tier-2 Merge-In verify failed once on `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` (`internal/loomcli/smoke_starttail_test.go`): the second `lyx loom start` got `index.lock: File exists` on the pair's weft worktree, so some process the first start left behind was still holding the records index.
The branch changes nothing in `internal/loomcli` or the records git path.
The test passed 5 of 5 times alone and the full tier 2 passed on the same HEAD, and no other gate log on record shows the failure.

The test's repeated starts do not wait for the records index lock the way `TestSmokeBootstrapLifecycle` does with `waitRecordsIndexLockGone`, so the flake will come back under load until that test waits too, or until `loom start` tolerates a concurrent records commit.
