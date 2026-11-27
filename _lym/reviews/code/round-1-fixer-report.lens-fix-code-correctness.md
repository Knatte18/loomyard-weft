```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code and tests the fix changed (fix-code-correctness). I traced it from context and ran nothing.

**1. MEDIUM — `internal/dobbyengine/runwatch_test.go`, row `AChildWrittenAfterTheListingWaitsForTheNextTick` (F8): it passes with the defect in place.**

- The first `w.tick()` reads the child and sets `watch.ChildSeen` to `w.childWritten` and `StatusSeen` to `watchWritten`.
- The row then takes `listed := w.childWritten`, which equals `ChildSeen`, writes `blocked`, and ticks with `ChildStatusModified = listed`.
- In `readWatchedRun`, `written := watch.Last == nil || !run.StatusModified.Equal(watch.StatusSeen) || !run.ChildStatusModified.Equal(watch.ChildSeen)` is therefore false. The child is never read on that tick, and the new `found &amp;&amp; !child.Modified.Equal(run.ChildStatusModified)` branch never runs.
- With that branch removed, the row behaves the same: no line on the second tick, then one `halted; … child blocked` line on the third. So F8's main path, keeping the last child when `Last != nil`, has no test.
- **Change:** make the listing see a write that was not read yet. After the first `w.tick()`, call `w.writeChild("running", "Plan-Write")`, then `listed := w.childWritten`, then `w.writeChild("blocked", "Plan-Write")`, then the tick over `listed`. This is the review's own premise that the child status was also written since the previous tick.

The sibling row `AFirstReadingOverAnOlderListingKeepsNone` does reach the branch, because `Last == nil`. Its `watch.Last != nil` assertion fails with the defect in place, but it covers only the first-reading arm.

**No other findings.**
- The remaining changed branches hold on the inputs I traced:
  - the torn read and the `PairDir` handling in `readWatchedRun`;
  - `compared()`;
  - `readNoticeChild`'s not-found returns (its `!found` callers in `runnotice.go` and `waitreminder.go` ignore the value);
  - the `errors.ErrUnsupported` arm;
  - the NaN and infinity refusal.
- The other added rows I traced each fail with their defect in place.

Tool calls made: none.
