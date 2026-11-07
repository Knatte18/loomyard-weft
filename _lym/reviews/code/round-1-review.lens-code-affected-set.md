```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set I found**

1. **Removed `wantHeldAtMost` field and its check.**
   - The set is every reader and writer of the field. My earlier `grep -rn wantHeldAtMost internal` returned nothing after the diff. The struct, the ceiling row and the assertion loop are the only places it appeared, and the diff updates all three. Covered.
   - The `wantHeld` field comment was reworded to state the new upper bound, so it matches the assertion at `startup_test.go:1946`. Covered.

2. **Shared upper bound `elapsed >= wantHeld + apiErrorRetryBackoff` on every succeeding row.**
   - The set is every row of `TestStartup_LoadTurnAPIError` that takes the success path. The test has no other consumers.
   - Rows that leave `wantHeld` at zero now get a 30s ceiling. These are the new open-turn row and the "turn start read during the back-off" row, which has its own `wantHeld`.
   - The ceiling row's bound is 1050s + 30s. The schedule in `apiErrorRead` (30, 60, 120, 240, 300, 300) sums to exactly 1050s, so its slack is the poll intervals.
   - The other succeeding rows, with `wantHeld` of 30s, 90s and 210s, have the same one-back-off slack. I found no row the new bound contradicts.
   - Covered. Whether the slack is sound against real poll counts is the code-tests lens's question.

3. **New rows "an open turn past its timeout that then ends…" and "an open turn after a hold fails the start at the deadline the hold moved".**
   - Both use existing fixture fields: `lateEnds`, `hangs`, `turnEnds`, `wantFailedAfter`, `wantHoldOver`. No fixture or fake in `fakes_test.go` changed, and none needs to.
   - `skillReed.playLoad` writes `START` for a late or hung turn only when `OpenTurns` is set. The table always sets it, so the new rows run as open turns.
   - Covered.

4. **Docs and rules that describe the hold.**
   - `doc.go:496`, the `loadturn.go` doc comments and the summary describe the production behaviour. That behaviour is untouched, so none of them goes stale.
   - No test elsewhere asserts the removed field or the old 25-minute bound.
   - Covered.

5. **PATTERN entries and commit message.**
   - The diff is one `_test.go` file, so no `PATTERN.md` or `pattern/` change is declared or needed.
   - `PATTERN-test-speed` is met: tier0, no sleeps, a fake clock, and the existing "not parallel because logger is process-global" comment still applies.
   - `PATTERN-test-isolation` is untouched.
   - The commit message carries no `Pattern-Departure:` trailer and needs none.
   - No stencil is changed.
   - Covered.

**Findings:** none within this lens.

One note outside the lens, for the handler. `_lym/landing/summary.md` says the new row fails the start "only at the run's deadline moved by the hold". The fail path at `startup_test.go:1932` checks only a lower bound (`elapsed < wantFailedAfter`). A deadline moved too far would still pass that row.

**Tool calls:** none. My context already held everything the procedure needed, including the earlier `wantHeldAtMost` grep.
