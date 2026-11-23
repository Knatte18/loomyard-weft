```yaml
lens: fix-affected-set
outcome: reported
```

Task: the affected-set lens for F1–F5 — rerun each finding's search and check the fix covers every member.

No findings blocking the fix. Per finding:

- **F1 (refusal sites for a note as a dependency):** a repo-wide grep for `cannot be depended`, `depends-note`, `CheckDependsNote`, `depends on note`, the old demotion-refusal text and the Sketch-refusal doc phrases returns nothing. Every site is covered.
- **F2 (callers of `RemoveTask`):** the production callers `boardengine/board.go:286`, `boardcli/cli.go:793` and `fronttask/hub.go:48` all reach `Store.RemoveTask`, which now strips. `MergeTasks` at `store.go:1314` calls `removeEntry` directly. The Bolt replay's own `replayRun.removeEntry` (`boardengine/replay.go:757`) is not covered: it still refuses a removal that leaves a dependent (`replay.go:766-770`), so it never leaves a dangling slug. Only the local `RemoveTask` path was in scope.
- **F3 (docs saying a note cannot be depended on):** the same grep finds no leftover statement in `.go`, `.md` or `.yaml` files.
- **F4 (duplicate sibling row):** the sibling row is gone from `siblings_test.go`, and no other duplicate came up.
- **F5 (the `BriefTask` doc comment):** applied.

- **LOW, `internal/boardengine/replay.go:757-771`:** the replay's removal still refuses on any dependent, while the local `RemoveTask` now strips.
  - The two paths disagree.
  - A replayed `lym board remove` of an entry that a note or task depends on is reported refused, although the same command succeeds locally.
  - Evidence: `replay.go:768` ("dangling dependency") against `store.go:761`.

No further tool calls were needed beyond the ones below.

- `grep` across `.go`, `.md`, `.yaml`, `.js` and `.json` for the F1/F3 stale phrases: confirms no site was missed.
- `grep` for `RemoveTask`, `removeEntry` and `RemoveEntry` in non-test code: lists the F2 callers.
- Grep for `KindNote` in `replay.go`, `promote.go` and `layers.go`: no extra stale note rule found. Not in my context.
- Read of `replay.go:752-780`: the replay removal path, not in my context.
