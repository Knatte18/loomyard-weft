```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for the fixed findings (F1, F4, F5, F10, F11) and check the fix covers every member.

Unchecked: F2, F3, F7, F8, F9 (each is one test row, no set). F6 is a decision-class finding and goes to the parent, so I did not check it.

Findings:

1. **LOW**, `internal/landingshed/prreview.go:414` (`PRReview.pause`), sibling of the F4 premise. It has the same defect F4 fixed in `parentlock.Waiter.pause`.
   - The `<-changes` case does not check whether the channel is closed. A closed channel ends every pause after the floor and loses the doubling backoff to the one-minute ceiling.
   - `doc.go:10-12` describes that backoff ("doubling to a minute while nothing changes"). The review named only `parentlock/acquire.go:76`, so the fix did not touch this member.
   - Whether the channel can close depends on `PullRequest.RecordChanges`, which I did not read. If it can, the same fix applies.
   - The fix did not make this worse, so under the rubric's third rule it may not count as a finding. I report it so the parent can decide.

2. **LOW**, `internal/landingshed/doc.go:44-60` (changed text: `doc.go` lines 53 and 55 and the `finalize.go` comment). No finding on the F6 sentence, which sits as the parent directed. Checked: after the F1 change, `doc.go:53-54` says the attempt record "keeps the HEAD at which the fixer's change was last let through", which still holds. A turn that leaves HEAD at Baseline now also records Judged, and the `judgeFixerChange` doc comment says so.

Searches that came back clean:
- **F4, callers of `parentlock.Waiter.pause`:** the only caller is `acquire.go:156`, which the fix updated to the two-value form. The only other use is the new test at `acquire_test.go:185`. The other `pause` matches belong to unrelated types.
- **F11, "oldest live ticket" and "queue head" wording:** `parentlock/doc.go:27`, `store.go:107` and `store.go:251` all now say "first live ticket". No stale wording remains.
- **F1, `Judged` writers:** every site that sets `Judged` goes through `letThrough` (`finalize.go:837`) or `rebaseAfterParentMerge` (`:747`). `proceed()` is called only from the `Baseline == ""` branch and from `letThrough`. No route lets a change through without recording `Judged`.
- **F10, `fswatch.Watch` sites:** `beamcli/dobbywatch.go:88`, `loomcli/landingdeps.go:372`, `reedengine/watchloop.go:15` and `shedverbs/loopwait.go:341` are outside the finding. Only `parentlock.Acquire` opens a watch, so `Waiter.Watch` covers that finding's set.
- **F5, `Call` loop:** `Call` is the only caller of `landUnderLock`. The front-start rule applies in one place.

Calls made:
1. `grep -rn "\.pause("`, to list every `pause` caller and sibling. My context lacked callers outside the files read.
2. `grep -rniE "oldest (live )?ticket|queue head"`, for stale wording. My context lacked the other doc files.
3. `grep -n "proceed()"` in `finalize.go`, to confirm the `proceed()` branches. Line numbers had shifted after the edits.
4. `grep -rn "Judged"`, to list every writer and reader of `Judged`. My context lacked other files that might write it.
5. `grep -rn "fswatch.Watch("`, to list watch-open sites for F10. My context lacked other packages.
6. `grep -n "func (p .*) pause("` in `prreview.go`, with `doc.go` Judged lines, to inspect the sibling `pause`. My context lacked `prreview.go`.
