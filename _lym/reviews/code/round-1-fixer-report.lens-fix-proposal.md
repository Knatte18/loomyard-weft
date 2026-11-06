```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 and F2 proposals as the fixer carried them out.

**Proposal F1** (post a line through `primeDeps` and `beamDeps` `WaitReminders.Post` and read it back signed DOBBY): no findings.
- The test runs a real in-process creel daemon, then reads its log for `posted` entries and compares `{from, to, body}` exactly.
- A wrong program name, a wrong scope slug (`<short>:task1:DOBBY` versus `<short>:DOBBY`), a wrong recipient or a swallowed error each fails it. The first run already showed that an unknown receiver surfaces as a `Post` error.
- `Post` runs once per dobby in a fixed order and `Log` returns entries oldest first, so the `slices.Equal` against two rows is deterministic.
- The daemon's socket and state come from a per-hub `CreelGeometry`, so `t.Parallel()` does not collide with other hubs' tests. `lymtemp.ProcessRoot` is read only.
- Subject is the first line of the body. The body is single-line, so comparing Subject to the full line holds. A multi-line reminder would need a different check.

**Proposal F2** (a cancel-marker row and an anchor-move row):
- The cancel-marker row sets the marker before the first tick, expects no post, and checks that `OperatorWait` stays empty. Without the `run.CancelMarker` skip at `waitreminder.go:68`, a post would appear at 30 minutes, so the row fails with the defect in place.
- The anchor-move row writes a real record into `geom.MoveRecords` before the first tick and asserts `awayReads == 0` over 31 ticks. If the `moving == ""` gate around `d.waitReminders` (`loop.go:611`) were removed, `Away` would be read each minute. `TestWaitReminders_CheckAtMostOnceAMinute` shows `awayReads` counts a wait check, so the row fails in that case.
- The record sits in a per-test `TempDir` and each subtest builds its own harness, so rows stay independent under `t.Parallel()`.

Tool calls: none; my context already held everything this lens needed.
