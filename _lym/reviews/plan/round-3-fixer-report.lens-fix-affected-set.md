```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I checked that the fix covers every member of each finding's set.

Findings left unchecked: none.

no findings

- F1's set is every place the after-tick read error is logged. I found no `d.failures` route for it anywhere in the plan. Card 8 now uses the `stateRead` episode, and `internal/dobbyengine#daemon` is already in the card's Edit list.
- F2's set is the after-tick read's outcomes:
  - marker gone: the rows that remove the worktree during a tick;
  - read error with the marker present: the directory row;
  - read succeeds with `Tidy` set or cleared: the tick-2 rows;
  - no record or undecodable: the new non-JSON row.

  "No record" and "undecodable" are one outcome (both read as no record), so the one new row covers both. No other card prescribes this read.

Tool calls:
- One grep of `_lym/plan/` for `d.failures`, `keptTidy`, `stateRead`, `after-tick` and `ReadJSONNoMkdir`. I needed it because my context showed only card 8, so I could not otherwise check that no other plan file uses the after-tick read or its error route.
