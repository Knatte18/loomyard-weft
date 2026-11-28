```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F10 proposal (wire the remaining PR-Review seams and record paths from `wired`) as carried out in `prreview_mail_integration_test.go`.

- LOW. Changed text: the file header line "The wired seams that read git, the pause record and the pull request record read a real task pair." and the `prReviewMailDeps` doc "The row mails, and reads git, the pause record and the pull request record, through the seams and record paths of wired."
  - The rewrite dropped `Mode` from both. The old text named "mode, origin and task head", and `pr.Mode = wired.PullRequest.Mode` is still assigned.
  - `Mode` reads `beam.yaml`, which is none of git, the pause record or the pull request record. Neither comment now says it is wired, although the test sets `beam.yaml` through `configengine.Set` for that seam.

I found no breakage in the proposal itself:

- **Shared record path:** the subtests now share the wired record file. They run sequentially, and each overwrites it whole with `WriteRecord`.
- **Real record watch:** `pause()` may now take the `changes` branch and drop the sleep's error. It still waits for the sleep to finish (`&lt;-slept`), so the sleep count and the fake clock advance as before.
  - The clock has moved by at least `prWaitFloor`, so `rest` is never positive and no extra sleep is counted.
  - A cancel dropped there is caught by the next wake's `cancelErr`, which wraps `ctx.Err()` with `%w`, so `errors.Is(err, context.Canceled)` still holds.
- **Branch names:** the wired `TaskHasCommits` reads against `main`, while `deps.ParentBranch` and `rec.Base` are `fixture-parent`. That mismatch already existed for `Origin`, and I found no assertion it affects.

Tool calls:
- One `grep` for `cancelErr` and `fail` in `internal/landingshed`: their bodies were not in my context, and I needed to know whether a cancel dropped by `pause()` still surfaces as `context.Canceled`.
