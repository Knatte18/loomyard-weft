```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 proposal, the rollback of the baseline move in `backUpBaseline`, and build cases where it is wrong or breaks something.

Cases I built and ran by hand:
- **Metadata rename fails, rollback succeeds.**
  - `os.Rename` of a directory is all-or-nothing, so a failed `baseline.meta` move leaves it in place.
  - `baseline` is absent at that point, so the rollback rename to `baselineDir` cannot collide.
  - The capture lock is held, so nothing else can create `baselineDir` in between.
  - The baseline and its metadata end up together, and the next capture takes the non-empty branch and retries. The proposal holds here.
- **Backup path conflicts.** `nextFreeBackup` guarantees `backup` and `backup.meta` are both free, so the rollback does not touch an unrelated directory.
- **Callers.**
  - `RunCapture` still calls `failCapture`, whose `cause.Error()` now covers the joined error text.
  - `MkdirAll` is skipped on the early return, as before. After a successful rollback `baselineDir` exists, so the skip is harmless.
- **Rollback also fails.** Then `baseline` is still absent and `baseline.meta` is still orphaned, so the next capture deletes that metadata. The review's fix accepts this by joining the rollback error. It is not a regression, and no sound alternative exists without a tested rename failure.

Finding:

- **LOW**: the `backUpBaseline` doc comment overstates the guarantee.
  - Changed text: "when that move fails, the baseline directory moves back so the pair is never left half-moved" in `internal/driftengine/capture.go`.
  - The code holds "never half-moved" only when the rollback succeeds. When the rollback rename fails, the function returns a joined error with the pair still split and `baseline` absent. The next capture then removes the orphaned metadata through its `len(entries) == 0` branch.
  - A reader relying on the comment would miss this residual case. It would be accurate if it said the baseline moves back "when it can".

Further tool calls: none. My context already held the review and the changed `capture.go`.
