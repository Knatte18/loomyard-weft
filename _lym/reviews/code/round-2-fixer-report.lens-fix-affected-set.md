```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set lens. I reran F10's search for the `Deps` seams the PR-Review row reads on the tested paths, to check that the fix covers every member.

Unchecked findings: none. F11 is `decision`-class and held by the parent.

The five seams F10 names, plus the two record paths, are all wired. The members below are not.

1. **MEDIUM:** `prReviewMailDeps` (`internal/loomcli/prreview_mail_integration_test.go`): `DriftVerdict` and `EnginesRun` are still envkit's canned `("", nil)` and `(nil, nil)` closures.
   - The row reads both on this path at every open, un-ended poll, through `wake` → `syncText` → `pullRequestText` → `completeSummary` (`finalize_pr.go:133-146`), and again in `openPullRequest`.
   - `landingDeps` fills `DriftVerdict` over the real pair (`driftengine.Verdict(driftRecordsDeps(l))`, which reads the committed drift records), so the real module is in reach. This is the defect F10 names, on a seam it did not list.
   - `wired.EnginesRun` comes from `runner.EnginesRun`, and the test passes a zero `&amp;shuttleengine.Runner{}`. Check that it works on that runner before wiring it.
   - The new header line, "The wired seams that read git, the pause record and the pull request record read a real task pair", reads as though no such seam is left canned.

2. **MEDIUM:** same function: `CommitRecord` is still envkit's `return nil`.
   - The row calls it from `recordPullRequest` and `updateRecord` through `commitPRRecord` (`finalize_pr.go:225-230`), including when it records the mailed head.
   - The fix moved `RecordPath` and `RecordLockPath` into the real pair, so `wired.PullRequest.CommitRecord` (`pullRequestRecordCommitter`, over fabric) now has the file it commits. The no-op stands in for fabric's commit while the real one is in reach.
   - The real seam also pushes through the pre-push check, so check that it runs on this hub fixture.

3. **LOW:** same function: `BaseCommand` and `GateTiers` are still envkit's fixed answers (`verifygate.go:154-155`), although `wired` fills both from the hub's gate config. The review accepted the scripted gate command. These two may fall under the same allowance, but neither the test nor its comment says so.

Tool calls:
1. `grep` of the `deps.` and `pr.` seam reads across the `landingshed` row files: my context held `prreview.go` but not the session, verify-gate or parent-align files.
2. `grep` of `completeSummary`, `resolveWithParentRule` and `commitPRRecord`: my context did not hold their bodies.
