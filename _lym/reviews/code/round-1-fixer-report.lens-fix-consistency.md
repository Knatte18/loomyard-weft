```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I found stale twins of the changed sweep wording; the other wording changes have no stale twins.

**LOW — creel sweep prose still says unread mail always keeps the sweep running**
- Changed text: `internal/creelengine/sweep.go` `sweepSubjects` now leaves out unread program mail to a hub orch, and `releaseGoneHolder` skips it.
- Three twins were left as they were:
  - `sweep.go:36`, the `sweepLoop` comment: "sweeps while a strand holds unread mail, …".
  - `sweep.go:173`, the `sweepNeeded` comment: "reports whether a strand holds unread mail, …". `sweepNeeded` is just `len(sweepSubjects()) > 0`, so it no longer holds for that mail.
  - `internal/creelengine/doc.go:265`, the first sentence of "Removed holders": "A sweep runs while a strand holds unread mail, …".
- The doc.go exception sentence is added three lines below, so a reader of the first sentence is briefly misled. The mismatch is small, and the new test (the program-mail row of `TestSweep_RemovedHolders`) shows why it matters: it posts a second message to keep the sweep alive.
- Fix: add "other than program mail to a hub orch" to those three sentences, or point them at the exception.

I found no other stale twins:
- The old `landedNotice` text appears only in `cli.go` and the updated `verbs_test.go`.
- The prune comments in `boardengine/board.go` and `store.go` already mention the `notify_on_land` exception.
- The `prune` Short, Long and flag text now agree with each other and with the docs edits.
- The `OwnedElsewhere` and `ClearNotifyOnLand` comments match the code.

Tool calls made:
1. `grep` over `*.go` and `*.md` for the old notice, prune and sweep wordings: my context lacked a repo-wide search for twin sentences.
2. `sed`/`grep` on `internal/boardengine/store.go` and `sweep.go`: my context lacked the `Prune` and `sweepNeeded` comments.
