```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the change set still agree with the sibling text it touches?

No findings.

- **Card 6 `TestRemove` row:** the goroutine `Settle` row says `Settle` reads the entry and the record both present or both gone. That matches card 6's lock design, where `Settle` either reads the record still there or reads the entry gone with no record. It also matches the `RemoveEntry` seam in `Deps`. No other card refers to this row or to a `Settle` called from inside `RemoveEntry`.
- **Card 11 `TestAdopt` row:** the row uses a diverged records branch of an adopted beam's pair. That branch qualifies for a warning under the D2 rule, because its `Slug` is an adopted beam and its state is diverged. The row also matches the worktree rule: `CodeWorktree` for a records branch. It does not contradict the replayed-records row, which keeps `ahead` with `Moved` set. No other card has `reconcileWarnings` rows that would need the same change.

Tool calls:
1. Searched `_lym/plan` for `RemoveEntry`, the lock-row wording, `reconcileWarnings` and `CodeWorktree`, to find copies of the changed rows outside the two lines my context showed.
