```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check through the fix-code-correctness lens the new `TestCommitWeftAt_SeedFailureFailsBeforeStaging` in `internal/fabricengine/commitweftat_test.go` (commit 55b8abc3). It is the only code the fix changed.

**Finding 1 — MEDIUM — the test may pass with the defect in place**
- Changed text: `TestCommitWeftAt_SeedFailureFailsBeforeStaging`, which makes `.git/info/exclude` a directory to force the seed to fail.
- Summary: git itself fails on that fixture, so the test may not tell the seed-failure branch from a missing one.
- Evidence: the first gate log shows `git status --porcelain` exiting 128 while `info/exclude` was a directory (`commitweftat_test.go:134`, log line 4390). `StageAllAndCommit` runs `git add -A`, which reads the same excludes file. It would likely fail the same way.
- Defect in place: delete the `seedWeftArtifactExcludes` branch at `recordsgit.go:376-378`, or swallow its error.
- Predicted result: `StageAllAndCommit` still returns a non-nil error, `committed=false` and an empty sha. The restored-file status is still `?? board-note.md`. All four assertions would pass.
- I did not run the defect case, since I'm read-only. It rests on the exit-128 observation.
- Suggested fix: use a seed failure git tolerates. For example, make `.git/info` a regular file. `MkdirAll` and the lock then fail, while `git add` ignores a missing excludes file. As a stronger check, assert that the failing error text contains `seed weft artifact excludes`, so a git-originated error cannot satisfy the test.

**Correctness of the success path**
- `newPlainWarpRepo` leaves a repo where `RemoveAll` plus `MkdirAll` on the exclude path is valid.
- With the fix as written, `mutateGitExclude` returns `read exclude file: ... is a directory` (not `IsNotExist`). `seedWeftArtifactExcludes` wraps it, and `commitWeftAt` returns it bare.
- Restoring a plain empty file before the status check is sound. Nothing was staged, so the status is exactly `"?? board-note.md\n"`.
- No signature changed, so no caller is affected.

Tool calls made: none. My context already held every changed file and the gate log excerpt.
