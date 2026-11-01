# Card 11 — board-replay-entrymerge

**Delete:**
- `internal/boardengine#replayField`
- `internal/boardengine#replayValue`

**Edit:**
- `internal/boardengine#replayFields`
- `internal/boardengine#replayRun.addEntry`
- `internal/boardengine#replayRun.changeEntry`
- `internal/boardengine#replayRun.removeEntry`
- `internal/boardengine#replayRun.changePrimeOwner`

**Uses:**
- `plan:internal/entrymerge#Table`
- `plan:internal/entrymerge#Field`
- `plan:internal/entrymerge#Table.Changed`
- `plan:internal/entrymerge#Table.Change`
- `plan:internal/entrymerge#Table.AddClash`
- `plan:internal/entrymerge#Table.Same`
- `plan:internal/entrymerge#Render`

**Intent:** Run the board's by-entry replay over the shared merge (D8), keeping around it what is the board's own.

- `replayFields` becomes an `entrymerge.Table[Task]` keyed by `Slug`, its field rows unchanged in order, getters and setters; the private `replayField` type goes.
- `replayRun.changeEntry` takes the changed fields from `replayFields.Changed(before, commit)` and the merged entry, the applied names and the losses from `replayFields.Change(base, before, origin, commit)`, appending each loss through `r.lost`; it keeps its own steps around that: the "origin does not hold" and "origin removed the entry" losses, the revised-date carry, the refused-dependency check, `normalizeTask`, `validateWrite` and the run lock.
- `replayRun.addEntry` reports a same-slug clash through `replayFields.AddClash`, and `replayRun.removeEntry` decides "origin changed the entry this commit removes" through `!replayFields.Same(origin, before)`; ids, handles, the draft flag, design files, `prime_owner` and the overview re-render stay the board's.
- `replayValue`'s callers, `changePrimeOwner` among them, call `entrymerge.Render`.
- Behaviour is unchanged: `TestFabricReplay`'s existing rows pass as they are, which is D8's criterion for the board, so no test changes; `boardengine/doc.go`'s replay paragraph is rewritten by card 12.

**ImpactSummary:** Internal to `boardengine/replay.go`: two private helpers removed and four `replayRun` methods delegate to `entrymerge`; no exported surface or result changes.
