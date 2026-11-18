```yaml
lens: guard-reach
outcome: reported
```

Guard-reach review: can each guard, branch and test row the cards add be reached from a real entry point, and does any guarded site recur unguarded?

**MEDIUM — Card 3, D2 failure path in case 2 (a runs strand plus a dead `loom-status` strand): the fallback is unspecified and no row reaches it.**
- Card 3 step 2 says that after a failed move, `ensureRunsStrand` reads the strands again and replaces a `loom-status` strand that is still present with the runs strand. It does not limit this to the no-runs-strand case.
- In case 2 a runs strand is already present, so that fallback replace would be refused.
  - The runs role is already held, and `ReplaceStrand` calls `validateNaming` before it removes anything (`internal/reedengine/replace.go`, `ReplaceStrand`).
  - The overview is held by a third strand, which reed refuses.
  - The real refusal would land as a second warning. The stale runs strand would then never get the keep or replace that today's `case runs != nil` arm gives it (`internal/orchcli/runs.go:84-87`).
- Every failed-move row in Card 3 starts from strand `s1` alone, with no runs strand.
- Fix: state that the re-read fallback applies only when there is no runs strand. In case 2, a failed move warns and the runs strand goes through its normal keep or replace. Add a row: a current or stale `r1`, a dead `s1`, and `replaceErrs` failing `loom-status`. It wants one warning naming `lym loom start`, and no replace of `s1` by the runs spec.

**LOW — Card 3, the fallback after a failed move: "a failed read there falls back to the add" has no reaching row.**
- `fakeStrands.Strands` cannot fail (`internal/orchcli#fakeStrands.Strands`, cli_test.go:62-68), and Card 3 adds no field to make it fail.
- In production, a failed re-read while `loom-status` still holds the overview makes the add hit reed's one-overview refusal. That lands only as the add warning naming `lym beam start`.
- The branch is reachable in production but untested, and its stated outcome (an add) fails in the very case where the move failed before removal.

**LOW — Card 5, the `ownRunLine` guard for "an unreadable status file draws no line": no row reaches it, and the Intent misstates it.**
- `internal/statuspane/cache.go` (`StatusCache.fail`) returns the last good status with `Found` true on the first failed poll. It returns `Found` false (`Unreadable`) only from the second failure on.
- So the line keeps drawing for one frame after a status becomes unreadable. The Intent says such a poll "then answers `Found` false", which holds only from the second failure.
- None of the sequence rows in `TestOverviewSource_DrawsTheOwnRunLine` writes an unreadable status.

**LOW — Card 5, the "`worktree` nil draws no line, as on a hand-built source" guard cannot be reached from any entry point.**
- `newOverviewSource` always sets `worktree` (Card 5). The `frame` tests build their sources through `newOverviewSource` (`internal/battencli#TestOverviewSource_DrawsTheOrchLine`, `#TestOverviewSource_DrawsTheDobbyAboveTheOrchs`), and no test or production code builds an `overviewSource` literal.
- Either drop the nil branch or call it a wiring guard. As written it claims a caller that does not exist.

**LOW — Card 4, the break-out naming row is reachable only through a fabricated state.**
- `moveShellWindowStrandsLocked` moves strand panes an older lym left in the shell window (`internal/reedengine/ownwindow.go:388-422`).
- No lym version set `Title` on a non-overview strand before this plan. So the "pawl" break-out row in `TestListReconcilePanesLocked_MovesShellWindowStrands` pins a state production does not reach today.
- An overview strand, whose title is something like `beam: prime`, that is broken out would now be named by its title instead of its role. That case is neither stated nor tested.

Further tool calls:
- grep for `"-n"` in the non-test files of `internal/reedengine`, plus `internal/statuspane/cache.go:10-90`. My context did not show whether any other site names a window (only `window.go:102` OpenWindow's temporary name, which is not a strand path), nor how `StatusCache.Poll` answers an unreadable file.
