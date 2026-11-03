```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1–F5 change set.

**MEDIUM — `internal/fabriccli/hubmutation_integration_test.go`, new step `ReconcileKeepsAKeyTheTemplateLacks`.** The step can pass with the defect in place.
- It plants `newer_board_key: 1` in `board.yaml` as an uncommitted change in the Bolt checkout, then runs `fabric reconcile`.
- `PullThenCommitWritten` is documented at `bolt.go:245`: "A Bolt that cannot be brought up to date, such as a dirty or diverged one, returns a non-empty skip". A dirty board makes the hub-wide heal skip with exit 0, and the file stays untouched.
- If `fabric.go:748` gained `PruneUnknown: true`, the skip would still keep the key, and the test would not fail.
- The test needs to commit the planted key to the board first, as the stale-hub test does with `gitkit.CommitFile`, so that reconcile runs and `PruneUnknown` matters.
- The existing step above it has the same weakness, but it is not part of this change.

**No findings** on the rest, each traced with one concrete input:
- `removeKey` in `internal/configengine/layers.go`: on `old:\n  inner: 1\nnewer_key: 3\n` with retired `old.inner`, the inner key is removed, the emptied `old` mapping is dropped, and the result is `newer_key: 3\n`.
  - Deeper chains unwind correctly, since each level checks its child on return.
  - A path that does not match returns `false` and the document comes back byte for byte.
  - The `DropRetired` caller at `config.go:230` and `pruneSparse` stay correct, because dropping an emptied parent is harmless for both.
- `TestDropRetired`: the "only child" row would give `old: {}\nnewer_key: 3\n` without the fix, so it fails with the defect in place. The other rows' expected bytes match the 2-space encoder output.
- Stale-build walk test: with `PruneUnknown` set on the walk, `newer_worktree_key` in the pair's `logger.yaml` would be removed and the `Contains` check would fail, so it pins the defect. It passed at tier 1 after the `darn` → `logger` switch.
- `configcli` tests: dropping `PruneUnknown` at `configcli.go:560` would leave `newer_key` in the worktree `loom.yaml` and fail the exact-bytes check. Dropping it at `hubwide.go:184` would leave `newer_loom_key` and `newer_landing_key` and fail the `gone` checks. Tier 1 passed.

Further tool calls (1 of 6):
- `grep PullThenCommitWritten` in `internal/fabricengine`: to see whether a dirty board skips the heal; the transcript only shows the doc comment line.
