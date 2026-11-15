```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, through the fix-affected-set lens, that the fixes for F1 and F2 cover every member of their sets.

Findings:
- F1 (set: the refusal sites in `settleFresh`). Covered. The grep of `refusal(` calls finds two before-spawn worktree sites, at `call.go:205` and `call.go:210`. Each now has its own message and its own `TestCall_Edits` row: `call_test.go:868` for the snapshot read and `call_test.go:872` for the changes read.
- F2 (set: every user of the `Changes` type). Covered. The grep finds these users:
  - the seam field (`edits.go`)
  - the `readEdits` compare (`edits.go:77`)
  - the attempt field and its settle site (`call.go:208` and `call.go:231`)
  - the real seam wiring (`loomcli/arbiter.go:192`)
  - the fake seam in `call_test.go`
  Every one of them uses `map[string]string`.
- Other callers of `fab.CodeWorktreeChanges()` (`gatecli/coveragescope.go:286` and `fabricengine/diff_integration_test.go:210`) still get `[]string`, which the fix did not change.
- No leftover `samePaths` references remain.
- LOW, not strictly a finding: `loomcli/arbiter.go:192` and the new `hashFileOrDeleted` in `drivewiring.go` have no test that reaches them. The only tests that touch `Changes` use the fake seam. The review did not ask for a test there, and `worktreeSnapshot` is tested the same way.
- Unchecked: no finding was left unchecked. I did not check whether an `attempt.json` already on disk, with `changes` as a list, still loads.

Tool calls:
1. A grep of `Changes` users and the `samePaths` leftovers across `internal`, `contracts` and `docs`, to find every member of F2's set.
2. A grep of `refusal(` calls and their tests in the arbiter, loomcli and battencli packages, to find every member of F1's set.
