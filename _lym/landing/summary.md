# Render HEDDLE.md in the hub config reconcile, not only on a heddle write

`HEDDLE.md` was rendered only by `lym heddle set`, `lym heddle remove` and heddle's Bolt replayer.
A hub with no heddle write since the overview landed had no file.
A file that existed went stale when a lym update changed the shipped entries it lists, because `heddle.json` stays as it was.

The hub-wide config reconcile now renders the overview in the Bolt checkout as part of its board write.
The reconcile runs once per lym build, so a hub gets a missing file, or a file a release made stale, with no hand step.
`lym heddle set` and `lym heddle remove` render it as before.

## Design decisions

- The render sits in `reconcileHubWide` in `internal/hubreconcile/walk.go`, inside the board write that already holds the Bolt write lock.
  It calls `heddleengine.RenderOverview`, which writes only when the bytes differ.
  A changed overview joins the board commit; an unchanged one adds nothing to it.
- A failed render is logged as a warning and leaves the file as it was.
  A `heddle.json` that does not load therefore never blocks the config reconcile.
- The overview's prior bytes are snapshotted with the config files, so a failed board write restores it.
  The snapshot read moved into a shared `snapshotFile` helper.
- `internal/hubreconcile` now imports `internal/heddleengine`; the seam allow-list in `seam_enforcement_test.go` is updated.
- The `heddleengine` and `hubreconcile` package docs and `pattern/PATTERN-heddle-instructions.md` name the reconcile as a writer of the overview.
- The heddle CLI integration test now expects the overview as the one modified tracked file in the Bolt checkout after a save.

## Tests

`TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview` in `internal/hubreconcile` covers four rows:
a missing overview, a stale one, one already current, and a `heddle.json` that does not load.
The missing and stale rows also run a walk whose board write fails first, and expect the overview restored.

## To check by hand

- The failed-render path in `renderHeddleOverview` (`internal/hubreconcile/walk.go`) is the one a review left open as untested.
  The "heddle store that does not load" row now exercises it; confirm that row asserts the warning path, not only the unchanged file.
- Nothing was run on a live hub: confirm that a hub with no `HEDDLE.md` gets one committed on the first walk after a lym update.
