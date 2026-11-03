```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set (F1–F6) against its surrounding code and docs for twins left unchanged. Finding list:

1. **LOW** — `internal/fabricengine/remove.go:1038-1054`, the nonzero-exit branch of `removeWarpWorktreeDir`.
   - The F4 fix stops a failed worktree listing from licensing a recursive delete, in `finishGitWorktreeRemoval` (`destroy.go`). The task-side nonzero-exit branch still calls `isRegisteredLinkedWorktree`, which answers false when `List` fails. A failed listing there still reaches `removeWorktreeLeftover` and deletes a worktree git refused to remove.
   - The review itself says "The task side already has this shape in `removeWarpWorktreeDir`", and the reviewer's Rejected list declined to route this branch through `finishGitWorktreeRemoval`. So the twin is a known and disclosed split, but the F4 hole is now closed on the sibling side and open on the task side. A reader of `remove.go:1093-1105` ("false, the conservative direction") will not see why the two sides differ.
   - Evidence: `remove.go:1038` `if !isRegisteredLinkedWorktree(l, target)` followed by `removeWorktreeLeftover` at line 1048, against `destroy.go:1414-1419` using `List` directly.

2. **LOW** — `remove.go:1050`, the task-side nonzero-exit message.
   - F5 adds the way forward ("stop whatever refills it, delete it by hand, then run remove again") to the clean-exit message at `destroy.go:1426`. The task-side message "... finishing its removal failed: %w" and the `destroy.go:1424` deregistered-then-failed message carry no such guidance.
   - The review notes the nonzero-exit path "shares the limit", so these are not regressions. They are still twin messages left as before, and `destroy.go:1424` is on the shared helper the fix edited a few lines away.

3. **LOW** — `destroy.go` doc comment of `finishGitWorktreeRemoval` against `remove.go:1089-1105`.
   - The doc on `isRegisteredLinkedWorktreeIn` still says a failure to enumerate answers false, "the conservative direction". The new `destroy.go` comment says an unlistable repository is treated as registered. The two are both right for their callers, but `isRegisteredLinkedWorktreeIn`'s caller list in its doc ("Remove ... Prune") is unaffected and not stale. This is a minor wording tension only.

Checked and consistent:
- `doc.go:589-590` matches both kinds of leftover the fix handles.
- The summary's test list matches the rows added: the task refilled-kept row, the `undeleted-kept` row, the `locked-sibling` row, and `TestAddRollback_FinishesRefilledWeftWorktree`.
- Nothing else in the repo pins the old clean-exit message text, so F5 breaks no test or doc. The grep found only the two `destroy.go` messages and the task-side one in `remove.go`.
- The new `listsLinkedWorktree` helper is used by both `isRegisteredLinkedWorktreeIn` and `finishGitWorktreeRemoval`.

Calls made:
1. Grep for the message wording across `.go` and `.md`, plus a read of `remove.go:995-1060` — the context lacked the `removeWarpWorktreeDir` body and any pin on the message text.
2. Grep of `doc.go` for the leftover paragraph — the context lacked the package doc to confirm it still matches.
