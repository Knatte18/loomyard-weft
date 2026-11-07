```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour: pinning test**

1. **Back-off upper bound.** The shared `elapsed >= wantHeld + apiErrorRetryBackoff` check (`startup_test.go:1946`) applies to every succeeding row. The fake clock advances only through `Sleep`, and the poll interval is the 1s `pollFloor` (`fastConfig`), so each row's real virtual time is its back-off sum plus a few seconds.
   - "each further API error doubles the wait" (wantHeld 210s, limit 240s): a tripled back-off gives 390s and fails.
   - "the wait stops doubling at the ceiling" (1050s, limit 1080s): a missing ceiling gives 1890s and fails. A ceiling of 6 minutes gives 1170s and fails. The schedule 30+60+120+240+300+300 equals the row's wantHeld, so the ceiling is pinned exactly to within the 30s slack.
   - "busy session" (wantHeld 90s, limit 120s): pinned.
   - Single-error rows (30s, limit 60s): a doubled first back-off fails.
   - Two gaps remain in the 30s slack:
     - A first back-off up to 59s passes the single-error rows.
     - A ceiling up to 5m29s passes the ceiling row.
   - These are LOW. They are not the tripled or large-overshoot cases the commit message claims to catch.

2. **Removal of `wantHeldAtMost` from the ceiling row.** The shared bound replaces it and is strictly tighter (1080s against the old 1500s). It is pinned.

3. **New row "an open turn past its timeout that then ends loads its skills".**
   - Mechanics: `OpenTurns` writes START at `playLoad`, and `lateEnds{0:8}` delays the turn end by 8 `Status` calls. At a 1s poll that is longer than the 50ms timeout, so the turn is held open past the timeout.
   - If `loadSkillTurn` timed out on an open turn (the `else if !now.Before(began+timeout)` branch taken despite `open`), the skills would be skipped and `skill skipped` logged. `wantSkips` is 0, so the row fails. It pins that the open turn is awaited.
   - `wantTyped` is "do the task" in both the right and the wrong case, so only `wantSkips` discriminates.
   - Duplication: the neighbouring row "a turn past its timeout that then ends on an API error" covers the open-turn-past-timeout path but ends on an API error, so it is not a duplicate. The "first turn that never ends is skipped" row in `TestStartup_SkillLoadTurns` has no OpenTurns, so it is not a duplicate either. I found no covering test (grep result below).

4. **New row "an open turn after a hold fails the start at the deadline the hold moved".**
   - It pins the lower side. If `heldDeadline` were not applied, the start would fail at about 1h (counted from the start), under `wantFailedAfter` of 1h+30s, and the row would fail.
   - The failure path has no upper bound, only `elapsed < wantFailedAfter` (`startup_test.go:1932`). A deadline moved by too much (for example the hold counted twice, or `began` set late) still passes, so over-movement is **unpinned**.
   - Overlap with the existing row "an open turn that never ends fails the start at the run deadline" (hold-free): that row pins only the unmoved deadline, so the new row is not a duplicate.

5. **Succeeding rows' deadline move.** `moved < tt.wantHeld` (`startup_test.go:1949`) checks only the lower side. An `endLoadHold` that over-extends the deadline passes every succeeding row.

**Findings**

- **LOW, class `design`.** Location: `startup_test.go:1932` (failure path), rows at 1804 and 1812. The failure rows check the fail time only from below, so a deadline moved by more than the hold passes. This is the shape focus item 4 asks about. Fix: add an upper bound such as `elapsed < wantFailedAfter + apiErrorRetryBackoff` for failing rows, or a `wantFailedAtMost`.
- **LOW, class `design`.** Location: `startup_test.go:1949`. The deadline move is bounded only from below, so over-extension is not pinned in any succeeding row. Fix: bound `moved` by `wantHeld + apiErrorRetryBackoff` as `elapsed` is.
- **NIT, class `consistency`.** Location: `startup_test.go:1732`. The `wantHeld` comment says "zero for a start that fails", but zero is now also the value for a succeeding start with no hold (the new open-turn row). That row's `wantHeld` of 0 makes the upper bound a 30s ceiling.

The remaining tool-call result is below.

Tool calls (1 of 6): none issued; the manifest context (`fakes_test.go`, `startup_test.go`, `loadturn.go`) was enough. I did not grep for other open-turn-ends-normally tests, so the "no covering test" statements in item 3 rest on the manifest only, not on a search.
