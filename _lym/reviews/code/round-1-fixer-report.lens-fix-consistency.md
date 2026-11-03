```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 change set, looking for twins left stale.

1. **LOW**: `internal/configsync/configsync.go:9` (package header, not edited this round). It still says an unknown key "leaves a base or worktree file only under the options' PruneUnknown". F5's fix narrowed the same rule in `PATTERN.md`, `PATTERN-config-layers.md` and the `ReconcileAll` doc to hub files, because a standalone worktree base ignores `PruneUnknown`. The header was left broad, so it now disagrees with the narrowed wording. The line comes from the earlier grep output, not a fresh read.

2. **LOW**: `internal/configengine/layers.go`, `removeKey` doc comment (not edited). It still reads as if it only deletes the entry at the path and reports whether one was removed. The F3 change also removes a parent mapping the removal empties. `DropRetired`'s doc was updated, but the helper's doc was not.

3. **LOW**: `internal/hubreconcile/ensure_integration_test.go`, the `runStaleBuildReconcile` doc comment (not edited). The test now also plants `newerWorktreeKey` in the pair's `logger.yaml` and asserts it survives. The doc comment still lists only the old assertions, and the prime's `logger.yaml` is not planted. This is cosmetic.

Checked and consistent:
- `PATTERN.md:21` and `PATTERN-config-layers.md` "Keys the template lacks" now match each other.
- The `doc.go` "carried in the result … logged when the walk writes the file" wording matches the F4 pattern wording.
- The `ReconcileAll` doc matches the `Options.PruneUnknown` doc.
- The `DropRetired` empty-parent removal is harmless to its other caller (`config.go:230`, the Load path). An emptied unknown parent is now dropped there instead of lingering as `old: {}`.
- The new configcli worktree subtest runs before the default-layer subtest. Later subtests overwrite or ignore the pair's `loom.yaml`.

Further tool calls: none. My context already held every file the lens needed.
