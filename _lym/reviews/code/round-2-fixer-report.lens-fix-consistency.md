```yaml
lens: fix-consistency
outcome: reported
```

Task: the fix-consistency lens over the F26, F2, F5, F13, F27 and F28 change set.

1. **LOW**: `internal/boardengine/store.go:340`, the `ParseBoardTasks` doc comment.
   - Summary: it still says "Each entry's retired pull_request and notify_on_land keys are folded as a load folds them."
   - Evidence: `foldLegacyBeamFields` now returns early for `KindProject` (`store.go:~287`), so a project's keys stay in `Unknown`. The `Load` comment at `store.go:189-191` and `boardengine/doc.go` were updated for this; the `ParseBoardTasks` twin was not.
   - Impact: only the comment is stale. `ParseBoardTasks` calls the same fold and behaves correctly.

Nothing else is stale in the change set:
- Nothing under `internal`, `pattern`, `plugins` or `docs` still names `LegacyBeamSettings`, `settingPlaces` or the old `Rewrite(existing, root, anchor)` signature.
- The `Rewrite` callers (`gateslot`, the `hubreconcile` fake, the `configreg` doc) all use the new signature.
- The `dobbyengine` and `hubreconcile` import allowlists match the imports they now use.

Tool calls:
1. `grep` over `*.md`, `*.yaml` and `*.go` for the old names and `notify_on_land`, to find stale cross-references outside the files already in my context.
