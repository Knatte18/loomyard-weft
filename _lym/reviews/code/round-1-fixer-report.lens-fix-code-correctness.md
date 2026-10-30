```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fixer changed for F1 to F4. I made no further tool calls, since my context already held every changed file and the gate results.

No findings.

- **`HeadHoldsCommitsNotIn` (F4).**
  - With the task level with its parent, `rev-list ref..HEAD` is empty and the method returns false. With one task commit it returns true.
  - A ref that doesn't resolve returns `false` and an error that names the ref.
  - Every call fails with a wrapped error. The `TaskHasCommits` closure returns the method's `(bool, error)` directly, so the old `len(commits) > 0, err` shape, which paired a value with an error, is gone.
  - `CommitLog` still has its caller, `CommitsMissing`, so nothing is left unused.
- **F1 test.**
  - Swapping the arguments in `CommitsNotIn` makes the "OneCommit" subtest fail, because `head..ref` is empty after a task commit.
  - A wrong parent ref makes the first subtest or the unresolvable-branch subtest fail.
  - A dropped error fails the third subtest.
  - The tier-1 loomcli run passed.
- **F2 rows.**
  - The PR-Review row's `Call` returns the `read whether the task branch has commits beyond its parent` error before any push or GitHub call, so `wantPushes` and `wantCalls` stay at their zero values.
  - The Finalize row reaches `errorStop` at `finalize_pr.go:121`, before any merge or GitHub call. Its `ParentMode` call precedes that, so the lock-held check still holds.
  - If either row dropped the seam's error, `Call` would return no error and `wantErr: true` would fail.
  - `wantErrText` is empty on every other row, and `strings.Contains(err, "")` is true, so no existing row changes.
- **F3 row.**
  - A `url.Error` wrapping `ErrTokenRejected` makes `tokenFailure` true in `Finalize.githubFailure`, so it skips `transientFailure` and halts with `LandingStopPrefix` and the token text.
  - With the old check, which only looked for an unresolvable token, the error would be read as transient. `Call` would then return an error and `wantErr: false` would fail, so the row would catch a revert.
- **Gate.** Tier 0 for landingshed passed 645 of 645 after the `fmt` import was added.

Process note, not a code defect: F2 and F3 landed in one commit because both touch the same two test files, while the fix step asks for one commit per finding.

Calls made: none.
