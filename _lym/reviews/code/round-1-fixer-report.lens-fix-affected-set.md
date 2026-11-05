```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run each finding's set search over the fix (F1 reflog subject, F2 record callers, F6 stale comments).

No findings.

- **F1:** `update by push` now has one declaration, `PushReflogSubject` at `internal/gitrepo/pushdestination.go:20`. It has no other use in Go or markdown outside `_lym` and `.lym`. `fabricengine` reads it through `gitrepo.PushReflogSubject`, and no `pushReflogSubject` or `trackingReflogMessage` remains.
- **F2:** Outside tests, `recordPush` has one caller, `recordLandedPush` (`push.go:179`). `recordLandedPush` is called from `push.go:172`, `:277`, `:308` and `:336`, which are `pushPlainWithRetry`, `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased`. `DeleteRemoteBranch` has one external caller, `destroy.go:1523`, whose `err == nil && deleted` check now sees the success the fix returns.
- **F6:** No `no push land`, `blocks every fetch and push` or `git's own for a push` wording remains. Other `push.autoSetupRemote` mentions survive and none is stale:
  - Test fixtures pass it as a `-c` flag (`shedtransient_test.go:31`, `publish_test.go:96`, `gitexec` transport tests).
  - The `toolfloor/doc.go:23` feature list names it.
  - `gitrepo/doc.go:183` and `pushdestination.go:25` use it only as an analogy for lym's own upstream setting.

Calls made:
1. `grep` for `update by push`, `pushReflogSubject` and `trackingReflogMessage` across Go and markdown, to rerun F1's set; the context held only the earlier Go-only grep from before the edit.
2. `grep` for `recordPush(`, `recordLandedPush(` and `DeleteRemoteBranch(` in non-test Go, to rerun F2's caller set after the edit.
3. `grep` for the stale-comment phrases and `autoSetupRemote` across `internal`, `docs` and top-level markdown, to rerun F6's set; the review named only three sites.
