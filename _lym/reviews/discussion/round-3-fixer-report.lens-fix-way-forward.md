```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every way forward the fix introduced: steps, rows, criteria and refusals.

**No findings.**

Each new path can be reached from the state it names and leads somewhere:

- **D1 exclude line:** it is seeded by `seedWeftArtifactExcludes`, which wiring runs and every fabricengine stage runs through `ensureWeftLockDirAt` first. So a temp file is never staged, whichever kind of pair it is in.
- **D5 kept task tip:** it is written before Finalize's first merge-in and kept on resume. The hand-merge criterion can be reached because Finalize writes the tip before its own `--ff` merge-in.
- **D5 GitHub merge commit:** `mergeOnGitHub` fetches the parent's upstream and fast-forwards the parent pair before the parent-side merge (`internal/landingshed/finalize_pr.go`, after the merge loop). So the recorded GitHub commit is an ancestor of the parent's head, and the record is read back. If it is not, the no-record rule's squash fallback still finds that commit.
- **D5 protected-branch criterion:** the way forward in `protectedParentWayForward` (merge on GitHub by hand, reset to origin, `lym loom resume`) leads to an up-to-date merge. The local merge commit in the record is then no ancestor, so the no-record rule finds the commit on origin's parent. That outcome makes progress.
- **D5 row with no landing commit:** this row is still written, and D6's listing now prints its reason, so the state can be read.

Tool calls:
- Read `internal/landingshed/finalize_pr.go:500-560`, to see whether the local parent holds GitHub's merge commit before the parent-side merge. My context showed only the start of `mergeOnGitHub`.
