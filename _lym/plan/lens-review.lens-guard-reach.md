```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. Two findings: one test row cannot fail against the old code, and the ending-window row never puts more than one overview in the fallback set.

1. **MEDIUM — Card 2, `TestFollowSessionVia` row `NoLiveAgentSelectsTheLowestIndexOverviewOfTwo` never reaches the new lowest-index branch.**
   - The row's listing is `@0 5 0 0 0\n@5 1 1 1 0\n@3 0 1 1 0` and it wants `@3`.
   - Today's `internal/reedengine#followTargetWindow` (`internal/reedengine/follow.go:73-76`) keeps the last overview it lists, which is also `@3`.
   - So the row passes whether or not the card changes the rule, and a regression back to "last listed" would go unnoticed.
   - Fix: list the lower-index overview first, for example `@3 0 1 1 0` before `@5 1 1 1 0`, so the old rule answers `@5`.

2. **LOW — Card 2, tier2 row `WithNoLiveAgentEndingTheViewedLowerOverviewSelectsTheOther`: the lowest-index choice cannot happen on this path.**
   - `internal/reedengine#Engine.RemoveStrand` runs `removeStrandLocked` before `killStrandPanes` (`internal/reedengine/strand.go:913-918`). The removed overview strand is therefore already gone from `st` when `selectRelevantWindowLocked` builds the overview set.
   - In this subtest only `pawl` is left, so the set holds a single window.
   - The old single-overview fallback also selects `pawl`. The row meets the D2 criterion ("never the shell"), but it does not prove the fallback picks the lowest-index window on this path. That choice only matters when a third overview window exists.

Further tool calls:
- grep of `internal/reedengine/reconcile.go`: to check whether reconcile kills a dead overview pane, which decides whether "its pane's exit" can end a window. It does not (`reconcile.go:119`), so Card 2's premise holds.
- `internal/loomcli/cli.go:225-250` and an `arm.go` grep: to confirm `start` arms (and so `resolveRecipe` fills `c.recipe`) before `ensureStatusStrand`. It does.
- grep for every `isOverview`, `AnchorOverview` and `overviewWindowOption` use outside the files the cards edit: to find a one-overview site no card guards. None was found.
