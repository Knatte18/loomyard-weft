```yaml
lens: fix-affected-set
outcome: reported
```

I checked both findings' sets: none left unchecked.

no findings

- F1 (the row where `Settle` waits on a lock its own goroutine holds): I searched all plan cards for rows that call `Settle` while a removal holds the lock, and for every fake `RemoveEntry`. Card 6's `TestRemove` row is the only such row, and the fix rewrote it to start `Settle` in a goroutine. Cards 0 and 7 also mention `Settle`, but neither has a row that runs it under a held lock or inside a removal.
- F2 (which worktree a warning names for each branch kind): `CodeWorktree` appears only in card 11, at line 79 for the rule and line 89 for the rows. The new `TestAdopt` row covers the records-branch case the review named. No other rule chooses a worktree by branch kind.

Tool calls:
1. grep of `_lym/plan/*.md` for nested `Settle`-under-lock rows and for `CodeWorktree`, to rerun each finding's search over the whole plan.
2. grep listing which plan cards mention `Settle`, and for `RemoveEntry` outside card 6, to find other cards that might contain the same row.
3. grep of the `Settle` lines in cards 0 and 7 for lock, removal or goroutine wording, since my context did not hold those cards.
