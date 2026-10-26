```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1, F2 and F3 fix changes through the fix-code-correctness lens (run each changed branch step by step, including call failures).

Findings:

1. **LOW** — `noticeDownDobby` in `internal/pawlengine/beamawait.go` records the notice after `r.notify(...)` whatever `notify` returned.
   - `notify` only logs a failure and returns nothing.
   - If it fails, the record is still written, and no later re-entry posts the notice during that down episode.
   - The review asked for "once per episode"; this makes a failed post permanent for the episode.
   - Making `notify` report success and recording only on success would fix it, but that change is optional given how the review framed the finding.

2. **LOW** — The comment on `awaitWake.acted` says "true when the wake answered a request".
   - F1 now also sets `acted` on `refuse`'s failed status write, where the request is kept and not answered.
   - The comment no longer matches the behaviour, and a reader would notice.

3. **LOW** — The new test row `ASecondReentryWithTheDobbyStillDownPostsNothing` pre-seeds the marker with `dobby-down-notified`.
   - No test exercises `recordDownNotice` writing the line after a first notice.
   - No test exercises `clearDownNotice` on a live dobby or in `refuse`.
   - A no-op `recordDownNotice` or `clearDownNotice` would pass every test, so those paths are unpinned.
   - The F2 `fix` key asked only for the second-re-entry row, so this is a gap beyond the minimum.

Walked through with no defect found:
- F1 `refuse`:
  - A failed `SetStatus` returns `acted: true`, so `Call` sets `wait` to `awaitFloor`.
  - The row's pinned `wantSleeps: seconds(1, 1)` would be `(1, 2)` with the old `awaitWake{}`, so it fails with the defect in place.
- First entry: `enter` writes the plain marker and calls no notice.
- Interrupted entry: `enter` skips `noticeDownDobby`.
- Down dobby, no record: `notify`, then `recordDownNotice` rewrites the marker with the timestamp and appends the line.
- Live dobby with a record: `clearDownNotice` rewrites the marker without the line.
- `refuse`'s clear: it runs after the request is removed and tolerates a read error.
- `BeamUp` still removes the marker at `beamup.go:53`, which clears the record for a new run.
- The new test row fails with the old code, because the old code would post a notice.

Tool calls: none; my context already held every changed file and the review.
