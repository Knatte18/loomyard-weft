```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the Notify closure the diff changes in `treeScopeSettings`.

**List: each function or branch, the case I ran, and whether it is correct**

- `treeScopeSettings`, darn branch, `Notify` closure, pair location. Input: `WorktreeName` is `budget-task` and the line is `Darn-Write gate: ... 30m0s budget`. `fmt.Sprintf` gives `loom budget-task: Darn-Write gate: ...`, which is passed to `orchcli.NotifyParent`. That function tags the mail with the same `WorktreeName` as its run (`internal/orchcli/notify.go:51`). **Correct.** The form matches raddle's notice (`internal/loomcli/raddlewiring.go:246`) and the seed notice (`internal/loomcli/wiring.go:995`, where `seedSlug` is the identity, `internal/loomcli/seedinput.go:77`).
- Same closure, `NotifyParent` fails (no live orch, or creel refuses). The error is returned unchanged, and `budgetClock.noticeIfSpent` logs it and does not retry (`internal/loomshed/verifybudget.go`, `noticeIfSpent`). `noticed` is already set, so the notice is not sent twice. **Correct**, and the diff does not change this path.
- Same closure, line at the boundary length. `verifyBudgetNotice` cuts the line to `verifyNoticeBytes` (1000 bytes), and the prefix is added after that cut. A posted subject can therefore reach 1000 + len(slug) + 7 bytes. I searched `internal/creelengine` and `internal/orchcli` for a cap on subject or line length and found none, so nothing refuses the longer line. **Correct**: the comment's aim of "one short creel line" still holds well enough.
- Same closure, empty `WorktreeName`. This cannot happen: `lymcwd.Location` sets `WorktreeName` from `filepath.Base` of a resolved worktree root (`internal/lymcwd/lymcwd.go:109`, `:226`). **Not reachable.**
- Same closure on the prime or a beam rather than a pair. `wireTreeScope` runs for every tree_scope recipe (`internal/loomcli/wiring.go:1339`), so the prefix would carry the prime's or beam's directory name. The seed notice at `wiring.go:995` behaves the same way, and `NotifyParent`'s run tag already uses that value. **Correct**, consistent with the existing notices.
- `treeScopeSettings`, hardener branch and error branches. The diff does not change them. The hardener branch returns a zero `VerifyBudget`, so no `Notify` runs. **Correct.**

**Findings**

None.

**Tool calls**

1. A grep of `internal/creelengine` and `internal/orchcli` for a cap on subject or line length. My context did not show whether creel refuses a line longer than the 1000 bytes loomshed cuts to.
