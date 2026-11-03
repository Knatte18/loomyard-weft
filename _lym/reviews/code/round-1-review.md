---
verdict: APPROVED
---

The diff `bd41b023..HEAD` makes the change the decision record settles and implements cards 1 to 4.

Checked against the focus directive, each with a verdict of not a defect:

- Placement: `overviewWindowInsertArgs` opens an overview window after the highest-index listed overview window whose key is not higher than its own, else before the lowest-index window. Equal keys sit in the order they last opened. A relaunch finds its own old window in the keys and opens after it.
- Persistence: `render.Display.Order` is `order,omitempty`, so an older display decodes as key zero. `overviewWindowKeys` reads it only for an overview strand.
- Refusals: the second-overview loop in `addStrandLocked` is gone, so `ReplaceStrand` no longer refuses either. `refuseWindowMove` refuses an overview order change and names remove and add.
- Focus: `relevantWindow` and `followTargetWindow` take the lowest-index overview window, and a live agent still wins.
- Runs strand: `ensureRunsStrand` no longer looks for a loom status strand. `RunsStrandOrder` is declared once in `orchcli`.
- Status strand: `statusStrandAddSpec` always returns an overview display, with the order derived from `RunsStrandOrder` and the recipe as Title. It reads no strands and branches on no recipe name. `c.recipe` is set before `ensureStatusStrand` runs.
- Docs: no statement of one overview strand per session survives in reed, orchcli, orchengine, loomcli, loomengine or `docs/overview.md`.
- PATTERN: the diff touches no PATTERN entry and no commit carries a `Pattern-Departure:` trailer. `PATTERN-named-constants`, `PATTERN-no-hard-coded-choice` and `PATTERN-loom-contract` hold.
- Tests: each acceptance criterion has a pinning row, at tier0 where the function is pure and at tier2 for placement and focus against real tmux.

## Rejected

- Lens code-correctness, LOW: a failed pane listing in `openOwnWindowLocked` leaves the key map empty, so a key-1 overview window can open before a key-0 one. Card 1 names this degradation ("which places before the lowest window as today"), the plan licenses it, and it needs a tmux listing failure.
- Lens code-tests, LOW: `AddedInTheOtherOrderTheOverviewsEndInTheSameOrder` would also pass if the add ignored keys. The key-aware branch is pinned by the tier0 `TestWindowOrder` rows, and the card prescribes this subtest, so no coverage is missing.
- Lens code-tests, NIT: the `overviewWindowKeys` subtest is a per-helper test. Card 1 prescribes it, and the helper's only caller is reached through tier2 alone, so no public-surface test at tier0 covers it.
