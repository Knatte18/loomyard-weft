```yaml
lens: fix-consistency
outcome: reported
```

No findings against the fixer's change set.

I checked these twin locations for drift:
- `internal/shedarbiter/doc.go` ("The fix", lines 31-44) says the call "snapshots the worktree at spawn, in the attempt record". That still holds, because `Changes` is recorded in the same `attempt.json`. It is a little less specific than the new `internal/loomcli/doc.go` sentence, but it is not wrong. It still says "must leave every change committed", which the untracked-path check now enforces.
- `docs/overview.md:1545-1546` says the arbiter commits on the task branch, a resume after its commits needs a goto, and batten edits nothing. None of that conflicts with the change. It is also consistent with F2, which is held for the parent.
- The fix fragment's line 22, "An uncommitted change, a switched branch or rewritten history hands the stop on", agrees with the new untracked-file check and the new "a new file included" bullet.
- `WorktreeSeam` has only two production and test constructors, `internal/loomcli/arbiter.go` and `fakeWorktree.seam()`. Both now set `Changes`, so no nil call is possible. The fabricengine `fakeWorktreeSeam` at `worktreeseam_integration_test.go:195` is a different type.
- The `conclude` change to `handOnWith(attempt, edits, ...)` makes the resume and agent hand-on paths consistent: both now put the problem and the commits in the question. The `handOn` doc and the `conclude` doc ("followed by the arbiter's commits and any problem with the worktree") still read correctly.

Tool calls:
1. Grep for snapshot and attempt wording across docs, contracts and the arbiter packages, to find stale twin descriptions of the record.
2. Read `internal/shedarbiter/doc.go:25-55`, because the package doc's "The fix" section was not in my context.
3. Grep of `docs/overview.md` for arbiter, commit and fix wording, to check the overview against the change.
