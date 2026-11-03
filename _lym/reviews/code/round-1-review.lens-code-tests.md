```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds or changes, the test that pins it (lens code-tests).

**Per behaviour → pinning test**

1. Placement after the last overview window with key ≤ own, else before the lowest window, else plain. Pinned by `TestWindowOrder` rows (key-1 listed, new 0 gives `-b`; equal key gives `-a` after it; key between gives `-a` after the lower), the two existing no-key and no-window rows, and the tier2 `TestWindowOrder_RealTmux` subtests.
2. Equal keys sit in the order they last opened (D1). Pinned by the `TestWindowOrder` equal-key row and the tier2 `EqualKeysSitInTheOrderTheyLastOpened`. Covered.
3. Add in the other order, relaunch and replace keep the order (D1). Pinned by tier2 `AddedInTheOtherOrderTheOverviewsEndInTheSameOrder` and `RelaunchingOrReplacingEitherOverviewKeepsTheOrder`. Covered.
4. Replacement that ends the viewed overview selects its new window. Pinned by the end of `RelaunchingOrReplacingEitherOverviewKeepsTheOrder`, which asserts `currentWindow` is the new window. Covered.
5. An agent opens after both overviews, and the shell and hidden windows stay after the agents. Pinned by the tier2 `AnOverviewWithAHigherKeyOpensAfterTheFirstThenAnAgentAfterBoth` and its `wantOverviewTitlesFirst` helper. Covered.
6. The second overview keeps every property of the first (own and overview markers, no unseen mark, no color, not selected). Pinned by the marks and `currentWindow` assertions in the tier2 `AnOverviewWithAHigherKeyOpensAfterTheFirstThenAnAgentAfterBoth`. Covered.
7. `overviewWindowKeys` skips a non-overview strand, a strand with no pane and a pane the listing lacks. Pinned by the `TestWindowOrder` subtest `overviewWindowKeys maps…`. Covered.
8. The empty `windowByPane` fallback when the pane listing fails. Pinned by none. It is an untested but trivial branch: the nil map reads as no keys, so the window goes before the lowest. Unchanged-in-kind from today, so I leave it unflagged.
9. `Display.Order` omitempty, and an absent key reads as zero (D5). Pinned by `render` `OverviewWithNoOrderKeyReadsKeyZero` and by the zero-order and order-1 round-trip assertions in `TestDisplayMarshalsWithoutRetiredKeys`. Covered.
10. The second-overview refusals are removed from `AddStrand` and `ReplaceStrand`. Acceptance is pinned only indirectly: the tier2 subtests add and replace overview strands beside another overview strand. The unit refusal test was deleted, so a reintroduced refusal fails tier2 only. Covered at tier2.
11. `UpdateStrand` refuses an order change, naming remove and add, and accepts a focus-only change. Pinned by `TestUpdateStrandLocked` `OverviewOrderChangeRejected` and `OverviewFocusOnlyChangeAccepted`. Covered.
12. D2: `relevantWindow` takes the lowest-index overview and a live agent still wins. Pinned by `TestAttachArgv_SelectsTheRelevantWindow` (fixture with two overviews, live-agent rows kept). Covered.
13. D2: `followTargetWindow` takes the lowest-index marker window. Pinned by `TestFollowSessionVia` `NoLiveAgentSelectsTheLowestIndexOverviewOfTwo`. Covered. A live agent winning over two overviews is already pinned by the existing `NewestLiveAgentWinsOverOlderAgentDeadNewerAgentOverviewAndNonOwnWindow`, which has only one overview.
14. D2: a viewer on the ending lower overview moves to the other overview, never the shell. Pinned by tier2 `WithNoLiveAgentEndingTheViewedLowestOverviewSelectsTheNextLowest`, which covers the removal path. Covered. The pane-exit path is not covered, but the card argues it cannot end a window (reconcile keeps dead own-window panes), so I raise no gap.
15. D3: `statusStrandAddSpec` always returns an overview display with the order after the runs strand's and the recipe as Title. Pinned by `TestStatusStrandAddSpec`, including the `> orchcli.RunsStrandOrder` assertion. Covered.
16. D3: `ensureStatusStrand` passes `c.recipe`. Pinned by the tier2 smoke subtest, which asserts the window is named `tapestry`. Covered. This is not pinned at tier0, but the plan puts it at tier2.
17. D4: the runs strand is added beside a loom status strand and the status strand is left untouched. Pinned by `TestStart_EnsuresTheRunsStrandBeforeTheOrch` (the renamed row plus the new no-replace, no-removal assertion, and `wantDisplay` with `Order`). Covered.
18. D4: a current runs strand is kept and a stale one replaced. Pinned by the existing rows, including the stale-runs rows beside a loom status strand. Covered.
19. D5: an own-window status strand is replaced into the overview window at the next start. Pinned by the tier2 smoke subtest (new guid, `@lym_overview`, window index one above the runs window). Covered.
20. `RunsStrandOrder` is a single named constant used by orchcli and loomcli. Pinned by `TestStatusStrandAddSpec` using the constants and `wantDisplay` using `RunsStrandOrder`. Covered.

**Findings**

- LOW, class `design`, `internal/reedengine/windoworder_integration_test.go:465-475`. Test economy and weak oracle: `AddedInTheOtherOrderTheOverviewsEndInTheSameOrder` re-adds `beam: prime` with key 0 while `pawl` is key 1. It would also pass if the add placed before the lowest window regardless of key. The keys-aware branch is covered by the tier0 rows, so this is not a coverage gap. Judge by the decision record's wording; no change is needed, so I do not raise it as BLOCKING or MEDIUM.
- NIT, class `consistency`, `internal/reedengine/windoworder_test.go:588-605`. The `overviewWindowKeys` subtest is a per-helper test, though `openOwnWindowLocked` reaches the helper only through tier2. The card mandates it and the helper is pure, so it is not a duplicate of a public-surface test. No action.

I found no behaviour left without a pinning row.

**Tool calls**
1. Read `/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYMHUB-pawl-checklist-hidden-in-beam/b88126f0-4505-40ce-86a6-b5c7fe59ca73/tool-results/b9aalniv6.txt` lines 549-668: my context lacked the tail of the `windoworder_test.go` diff.
