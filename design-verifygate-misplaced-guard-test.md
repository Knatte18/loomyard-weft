# Restore the verify-gate test for a misplaced //lym:guard marker

`verifygate-misplaced-guard-test` · Task · enhancement, loom, gate · running · Code-Burler-Fix

The verify-gate integration test in `internal/loomshed/verifygate_integration_test.go` had a subtest, `fails with the file and line of a misplaced marker`, that always skipped: the gate key's call carries no `--base`, so a narrowed run never reaches the guard scan over the diff.
Task `test-skips-os-tags` deleted it, since no test is skipped by design.

Once the key reads its diff base from the green record, add the case back: a committed `a/a_test.go` with `//lym:guard` above a non-test declaration fails both the loom gate and the darn gate, naming `a/a_test.go:5` and the way to place the marker.
