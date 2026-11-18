```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set, checking that F17's fix covers every site of the "stop's outcome" wording and every stop-ended return.

Unchecked findings: none. F17 was the only finding, and I checked it.

no findings

F17's set is every place in the plan that states what a stop-ended relaunch returns, plus card 16's matching return:
- **Card 14:** all three sites (line 65, step 1 at line 70, and the before-the-mark row at line 117) now carry the new shape. "The stop's outcome" no longer appears in any plan card; its only remaining hits are in earlier lens reports, which are not cards.
- **Card 16:** line 32 says its path returns as card 14's start path returns, so it picks up the stop-ended shape by reference. Nothing in card 16 states a stop outcome of its own.
- **New row:** the stop driven through `start` is added.

Calls:
1. grep `_lym/plan/` for "stop's outcome" and stop-ended phrasing, to rerun F17's search across every card; my context only held card 14.
2. grep card 16 for ErrNotStarted, outcome and stop, to check the sibling card that the review's Rejected section says F17 also covers; my context did not hold card 16.
