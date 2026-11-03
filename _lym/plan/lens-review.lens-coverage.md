```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the plan in `_lym/plan/`.

1. MEDIUM. D2's acceptance criterion on the pane-exit path is argued away, not tested. The criterion says a viewer on the lower-index overview window moves to the other overview window, never the shell, when that window ends "by its strand's removal or its pane's exit". Card 2 tests only the removal. It argues that "a pane that exits stays as a dead pane under remain-on-exit, so its window ends only at that removal or a relaunch". The code contradicts that argument: reconcile kills panes and ends own windows itself, then moves the viewer through `selectRelevantWindowIfCurrentLocked(st, plan.ownWindowsToEnd)` (`internal/reedengine/reconcile.go:256`, `:263`, `:271`). Card 2 should plan a row where an overview pane exits while it is viewed and reconcile ends its window. Either a tier0 reconcile row with two overview strands, or a tier2 subtest in `TestWindowOrder_RealTmux`. The row would assert the current window becomes the other overview window.

2. MEDIUM. Card 2's new `TestFollowSessionVia` row `NoLiveAgentSelectsTheLowestIndexOverviewOfTwo` would pass against the old code too. Its listing `@0 5 0 0 0\n@5 1 1 1 0\n@3 0 1 1 0\n` puts the lowest-index overview `@3` last. The old `followTargetWindow` keeps the last overview line it reads (`internal/reedengine/follow.go:75-76`), so it also answers `@3`. The D2 rule for the follow target is therefore never tested. Listing `@3 0 1 1 0` before `@5 1 1 1 0` fixes it, because the old rule would then answer `@5`. Card 2's attach fixture already does this the right way.

3. LOW. The D1 criterion says an order-key update is refused "naming remove and add". Card 1's row `OverviewOrderChangeRejected` checks only `way forward: lym reed remove`, the existing `moveWayForward` constant (`internal/reedengine/strand_test.go:264`). A message that dropped "then add it again" would still pass. The row should also check the add half.

4. LOW. D5 says the heal happens at a loom start "whose sidecar names another build". Card 4's smoke subtest pre-adds the own-window status strand with no sidecar at all. Both cases take the same replace branch (`internal/loomcli/bootstrap.go:226-235`), but the plan never tests the case the record names. Writing a sidecar for another build in that subtest would cover it.

5. LOW. D3 says the status strand's key comes after the runs strand's. Card 4's `TestStatusStrandAddSpec` rows expect `Order: statusStrandOrder`, so they would pass even if that constant were wrong, for example equal to `RunsStrandOrder`. Only the tier2 smoke check that its window index is one above the runs window catches that. A tier0 assertion `got.Display.Order > orchcli.RunsStrandOrder` would test the rule at the lowest tier.

6. LOW. Card 1 adds a branch no row reaches. When the pane listing fails in `openOwnWindowLocked`, the keys map is empty and the window opens before the lowest-index window. Card 1 states that fallback but plans no row for it.

7. LOW. One of the record's constraints goes unchecked for the second overview window. The constraint is that an overview window carries `@lym_own` and `@lym_overview`, no `@strand_color` or `@lym_unseen`, and is never selected when it opens. Card 1's new tier2 subtests check placement only. The existing marks check covers only the first overview window (`windoworder_integration_test.go:233-238`). `AnOverviewWithAHigherKeyOpensAfterTheFirstThenAnAgentAfterBoth` could also read the `pawl` window's marks and check the viewer stayed where it was.

Tool calls:
- One grep of `internal/reedengine/reconcile.go`, to check whether reconcile ends an own window whose pane exited. That file was not in my context.
