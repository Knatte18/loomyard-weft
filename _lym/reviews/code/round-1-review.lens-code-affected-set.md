```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, find everything outside its lines that calls, reads or asserts it, and judge whether the diff left any of it wrong.

**Per change, the set found and whether it is covered**

1. **`newBattenIndex(src)` becomes `newBattenIndex(src, windowStart)`.** Its callers are `report.go:72` and `timing.go:181`, and the diff updates both. No test calls it directly. Covered.
2. **`selectRuns` gains a `batten` parameter.** Its callers are `report.go:76` and `timing.go:186`, both updated. The doc comment at `runs.go:131` is updated. Covered.
3. **`selectInFlight` gains a `batten` parameter.** Its caller `timing.go:190` is updated and the doc comment at `timing.go:238` is updated. `cost.go:120` calls `ComputeTiming`, whose signature does not change. Covered.
4. **Removed symbols.** `tasksAt`, `boardDoneAfter`, `labelsOf`, `commitsOf`, the `battenCommit` type and `boardTasks` leave no reference in `internal` or `cmd`. The remaining `battenCommit` hits are the unrelated `RunRow.battenCommit` field (`runs.go:79`, `timingstops.go:84`). Covered.
5. **`joinTime` and `assemble` no longer return an error, and `timeJoinOwners` and `windowOwner` move to package level.** The only caller is `readLandings`, which is updated. Covered.
6. **Behaviour: landing checkpoints now come from the bounded records walk.** Their reader is `timeJoinOwners`, through `report.inputs.checkpoints` (`runs.go:210`). `doc.go:48` states the bound, and so does the summary. Covered.
7. **Behaviour: the batten walk is bounded and widens on demand.** `doc.go:46-50` describes it. That text calls the walk a single one, which `widen` contradicts (finding 2).
8. **Behaviour: the board history is read once from a floor.** `doc.go:89-92` describes it, and `readLandings`' doc (`landings.go:117-119`) matches. Covered.
9. **New fixture state: `fakeRepo.fileReads` and `logQueries`.** The new tests read them. Existing tests are unaffected because the fields only count. Covered.
10. **The new reflective test `TestLandingJoiner_HoldsNoParsedBoard` and PATTERN-test-speed's `//lym:seam` rule.** The test is only partly marked (finding 1).
11. **CLI help text.** `reportcli/cli.go:88-99`, `cost.go:36` and `raddle.go:22-30` make no claim about walk bounds. Nothing goes stale.
12. **Commit trailers.**
    - The `Root-Cause:` trailer on `d067dad6d` names the unwindowed walks and the caches that held every parsed revision. Those are the defects the diff removes.
    - There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed: no test at the review base is loosened.
13. **Declared PATTERN changes.** The directive declares none, so nothing is checked.

**Findings**

1. **BLOCKING, design.** `internal/reportengine/landings_test.go:836-848` (`TestLandingJoiner_HoldsNoParsedBoard`): the reflective test lacks `//lym:seam` lines for most of the structs it walks.
   - **Evidence:** `pattern/PATTERN-test-speed.md:141-142` requires "one such line per struct its reflection walks, nested structs included".
   - `typePathTo` (`landings_test.go:853-878`) recurses through every struct field from `landingJoiner`. That reaches:
     - `Sources` and `Options`;
     - `Report`, and through it `RunRow`, `Landing`, `Finding`, `PartialRun`, `landingInputs` and `selectedTag`;
     - `gitrepo.LogCommit` and `gitrepo.TagRef`;
     - `shedengine.Status` and `HistoryEntry`;
     - `boardHistory`, `commitSpan` and `labelSpan`;
     - `landingClaim`.
   - The test carries only `internal/reportengine#landingJoiner` and `internal/boardengine#Task`.
   - **Fix:** add one `//lym:seam` line per struct the walk reaches.
2. **LOW, consistency.** `internal/reportengine/doc.go:46`: "A report walks the records branch once … from a week before its window" contradicts the widening path.
   - **Evidence:** `batten.go:110-127` (`widen`) walks again, sometimes twice in one call.
   - The "week before" is also measured from `landingWalkStart` (Since minus FreshDays) in `Compute` (`report.go:72`), not from the window's Since.
   - **Fix:** say the first walk opens a week before the landing walk's start and that each widening walks again.

**Tool calls**
- Grep for the removed symbols, `doc.go`'s walk lines and the docs or patterns naming them: the manifest held no repo-wide search for stale references.
- `doc.go` lines 30-95 and the report CLI help text: the manifest held only `doc.go`'s diff hunks, not the prose around them.
- Grep for the `lym:seam` scan and the nested-struct rule: to confirm the rule's text and where the scan lives.
