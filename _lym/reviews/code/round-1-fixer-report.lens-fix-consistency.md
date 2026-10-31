```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for consistency with its surroundings (fix-consistency lens).

**Findings**

- LOW, `internal/orchengine/watcher_compact_test.go`, `TestCompact_HookMarkStampedBeforeTheBoundaryIsNotTypedAgain`: the name is now stale. The function became a two-row table, and one row (`a mark older than the previous baseline types the pointer`) expects the pointer to be typed. The name and the first sentence of the doc comment still say the pointer is "not typed again". The doc comment's second line was extended, so the doc comment carries both behaviours while the name carries only one. The twin test in `watcher_test.go`, `TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped`, names both outcomes, so the two are inconsistent with each other. A name such as `TestCompact_HookDeliveredPointerIsSkippedAndAStaleMarkIsTyped` would match.

No other inconsistencies found:
- The new `TestWatcher_ClearCycleTypesThePointerWhenTheMarkPredatesTheBaseline` follows the `TestWatcher_*` naming and uses the same `setState` then `newWatcher` setup as its auto-compaction twin.
- The `PointerMarkSince: deferred` input and the zero assertion sit beside the other cleared-field assertions in the non-idle `TestResetForFreshLaunch` subtest, as the F3 fix asked.
- The three `PointerMarkSince` sites in `watcher.go` (lines 1233, 1273 and 1649) each now have a test row. The fixer's mutation run showed the clear and compact tests failing with the assignments removed.
- `watcher.go` was restored from the backup, and `git status` showed only the three test files modified.

Process note, outside the artifact: the three findings went into one commit (`318275cd`), but `instruction-fix.md` asks for one commit per finding.

Tool calls: none.
