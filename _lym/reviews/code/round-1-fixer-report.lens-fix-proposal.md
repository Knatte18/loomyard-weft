```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 proposal (a seed-failure test for `commitWeftAt`) as the fixer carried it out in `TestCommitWeftAt_SeedFailureFailsBeforeStaging`.

Proposal under test: make `info/exclude` a directory so the seed read fails, then assert a non-nil error, committed=false, an empty sha and an unstaged dirty file.

Cases I built against it:

- **Test fails with the defect in place.** If the seed error were swallowed, `StageAllAndCommit` would stage and commit `board-note.md`. Then `git status` would no longer report `?? board-note.md`. Either way, `err == nil` fails the first assertion. The test discriminates.
- **Cleanup.** `t.TempDir` removes the directory the test swapped into `.git/info/exclude`, and the test restores a plain file before the status check. Nothing is left behind.
- **Parallel safety.** The fixture is per-test (`newPlainWarpRepo`), so nothing is shared and `t.Parallel()` holds.
- **Platform portability.** Replacing a file with a directory works on Linux and Windows. `os.ReadFile` on a directory returns an error that is not `IsNotExist`, so `mutateGitExclude` fails the same way on both (`gitexclude.go:69-72`).
- **Gate run.** After the fixer's second edit, the tier-1 run of `./internal/fabricengine` was green (`failed 0`).
- **Assertion strictness.** The test requires `status == "?? board-note.md\n"` exactly. `newPlainWarpRepo` makes one commit and leaves no other untracked files, and the restored empty `info/exclude` adds none. The assertion held in the green run.
- **Path assumption.** The test hardcodes `.git/info/exclude`, which holds only because `newPlainWarpRepo` is a plain `git init` repo. `resolveGitExcludePath` is exercised against that same layout, so the failure does hit the seed path.

No finding breaks the proposal, and I found nothing else it affects.

no findings

Tool calls made: none; my context already held everything this lens needed.
