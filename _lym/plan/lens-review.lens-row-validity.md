```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity — whether each planned test row reaches its branch and would fail if the code were wrong, plus test economy.

- **MEDIUM — Card 2, D2 follow target: the new `TestFollowSessionVia` row `NoLiveAgentSelectsTheLowestIndexOverviewOfTwo` passes under the old code.**
  The listing `@0 5 0 0 0\n@5 1 1 1 0\n@3 0 1 1 0\n` lists the lowest-index overview `@3` last.
  The current `followTargetWindow` (`internal/reedengine/follow.go:75-76`) keeps the last overview line it reads, so it already answers `@3`.
  No tier2 follow-session row covers two overviews either, so the D2 follow-target criterion goes unproved.
  Fix: list the lowest-index overview first, e.g. `@3 0 1 1 0\n@5 1 1 1 0\n`, so the old rule would answer `@5`.
- **LOW — Card 2, test economy: `NewestLiveAgentWinsOverTwoOverviews` repeats an existing row.**
  `NewestLiveAgentWinsOverOlderAgentDeadNewerAgentOverviewAndNonOwnWindow` (`internal/reedengine/follow_test.go:38-43`) already reaches "a live agent wins over an overview".
  A second overview line takes the same `own && !dead` branch. Fold it into the existing row or drop it.
- **LOW — Card 2, D2 viewer move: the tier2 subtest `WithNoLiveAgentEndingTheViewedLowerOverviewSelectsTheOther` passes under the old code.**
  After `beam: prime` is removed, `pawl` is the only overview strand left in state, so the old last-listed fallback in `agentWindowPanes` (`internal/reedengine/ownwindow.go:440-441`) also selects `pawl`.
  It pins "never the shell", not the lowest-index rule. The discriminating proof rests on the reworked `relevantWindowFixture` row, which does fail under the old rule.
  Run the subtest after `third` exists if it should discriminate.
- **LOW — Card 1, the new `overviewWindowKeys` subtest pins only exclusions.**
  It covers a non-overview strand, a strand with no pane, and a pane the map lacks. A function that always returns an empty map passes it.
  Add a positive row: an overview strand's bound pane maps to its window id with its `Order`.
- **LOW — Card 1, `TestWindowOrder_RealTmux` new subtests: the literal want lists don't match the session state at that point.**
  - The `plain` window opened at `internal/reedengine/windoworder_integration_test.go:150` is never ended. It is visible and not own, and the literal `beam: prime, pawl, shell, …hidden` omits it.
  - The helper `agentNames` (`windoworder_integration_test.go:202-211`) excludes only the name `overview`, so the titled overviews `beam: prime`, `pawl` and `third` would count as agents if reused.
  - The card should say to compute the want from the listing, overview titles excluded from the agents, not to write it literally.
- **Other (coverage lens):** the D2 criterion's "ends by its pane's exit" path is argued away in Card 2's Intent, not tested.
  That argument assumes reconcile never ends a dead overview pane's window, which this lens did not verify.

Calls made:
- Read `internal/orchcli/cli_test.go:30-100` (`fakeStrands`), `seedLLMDriver` and `tmuxBinaryPath`, a grep for `"overview"` window-name assertions in the loomcli, orchcli and battencli tests, and `internal/reedengine/strand_test.go:304-315`.
  My context lacked the fake's call recording for Card 3's no-replace/no-remove row, the smoke helpers' recipe for Card 4, the window-name assertions that could break, and `TestUpdateStrandLocked`'s accepted-path assertions for Card 1's new rows.
