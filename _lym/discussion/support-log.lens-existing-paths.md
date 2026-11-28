```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already read or write what the record's decisions change.

**Findings**

1. **BLOCKING, D2/D4: the new tapestry graph breaks the loom contract.**
   - Evidence: `internal/shedbuild/loomcontract.go:17,32` says "The contract is one record row" and calls `onlyRowOf(r, shedrecipe.LoomRecordEngines, "record", "DiscussionWrite or RecordWrite")`. Both engines are record engines, so a recipe may hold only one of them.
   - After D2 and D4, the expanded tapestry holds Discussion-Write from the Motif block and Record-Write from the Jacquard block. Shed's recipe loader would refuse to build tapestry at all.
   - The record never mentions the contract. A decision has to change it, for example to "exactly one record row on each on_done path from the entry, or a DiscussionWrite row followed by a stored-only RecordWrite". Or the design has to seed jacquard's record some other way. Either is a `PATTERN-shed-recipe-registry` loom-contract change and belongs in `## PATTERN changes`.

2. **MEDIUM, D2/D4: `lym loom decision add` on a tapestry run now has two record rows to choose from, and the record doesn't say which one counts.**
   - Evidence: `loomrecipe.RecordEngine` (`internal/loomrecipe/recipekeys.go:156`) returns the first record engine in row order. `decision.go:1724-1730` sends a run whose record engine is RecordWrite through `checkDarnRecord`.
   - Tapestry finds Discussion-Write first and stays on the Discussion path. That survives, but only by row order. Jacquard returns RecordWrite and takes darn's path, where `recordWritePending` reads Preflight and Record-Write as "before the write". That still reads right for jacquard's order.
   - The record should state that tapestry's decision calls read Discussion as the record row and jacquard's read Record-Write, and pin both with a test. Today a reorder or a contract change would switch the path silently.

3. **LOW, D4: the way forward for a tapestry run parked at Preflight may not work.**
   - It names `lym loom goto --to Motif`, and Motif is a block row. It is unverified whether goto admits a block row as its target, and whether a goto to it re-runs the undiscussed gate rather than its SkipTo.
   - Either verify it, or name the Discussion-Write row as the target.

4. **LOW, D2: the run checklist will show the build rows nested under a "Jacquard" block.**
   - Evidence: `internal/loomcli/checklist.go:293,436-437` names a row by its `BlockPath`. A tapestry run's build rows now carry `BlockPath` "Jacquard", so the checklist shows them as block children.
   - This is display only; on-disk row names are unchanged. The record could note it, or leave it as accepted.

**Paths traced, per decision**

- **D1**
  - `loomrecipe` embedded map and `parseEmbeddedRecipe` (Parse then Expand): survives.
  - `RecordEngine` for jacquard returns RecordWrite, so decision add takes darn's path with the right pending rows: survives.
- **D2**
  - Loom contract check: breaks (finding 1).
  - Motif gate's `seedDiscussedRecord` overwrite: unchanged.
  - `arm.go:334` `HasEngine(RecipeTapestry, …)` for a run with no seed: survives.
  - Checklist block naming: changes display (finding 4).
  - Report phase families keyed on `NameMotif` and `NameDiscussionWrite`: survive for tapestry; jacquard needs its own table, which D8 gives.
- **D3**
  - Expand's empty on_done refusal and the BlockPath readers (`routing.go`, `build.go`): survive given the last-row bound.
- **D4**
  - `recordWriteDeps.ReadStored` reads `board.DecisionRecord`, which is found only for a discussed entry: survives.
  - Tapestry's approval writes the record before Record-Write runs, so Record-Write writes nothing: survives.
  - The darn row is unconfigured and keeps its body fallback: survives.
  - The goto way forward: unverified (finding 3).
- **D5**
  - Batten's start guard reads the record file the same way raddle's does: survives.
  - A resume passes over the guard, and Record-Write's Stuck stays in force: survives.
- **D6**
  - Filing sets `Discussed` and copies `part.Recipe`: survives.
  - `raddlecontext.go:115` profile listing: D6 covers it.
- **D9**
  - `sharedbootstrap.go:245` sets `StoredRecord` from `Discussed`, so jacquard runs always record true and the cost key reads right: survives.

**Tool calls**

- `internal/loomcli/decision.go` 590-625, `wiring.go` 515-560, BlockPath grep: none of these were in context; I needed decision add's Record-Write branch and the readers of BlockPath.
- decision.go grep for `checkDarnRecord`/`HasEngine`: to find how decision add picks the record path.
- decision.go 1040-1065 and 620-660: to read the darn record branch and the pending-row ways forward.
- `rowWritten` wiring and `boardCopyDue`: to find the predicate that selects the darn path.
- `RecordEngine` and the loom contract's record-row rule: to check whether two record rows are allowed.
