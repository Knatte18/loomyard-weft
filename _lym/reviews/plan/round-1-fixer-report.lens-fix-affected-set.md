```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran each set-premise finding's search and checked that the change set covers every member, not only the ones the review named.

Unchecked: none of the set-premise findings (F3, F9, F11, F12, F13, F16 and F18). F3 is checked as a single-caller premise: card 1 now joins the path inside `websterengine.ConflictRecordRel` itself, and no new `runplan` function is left.

no findings

- **F12:** every place that puts the record under `.lym/webster` or a scratch directory is covered by card 1's Edit list:
  - `contracts/recipes/tapestry-recipe.yaml:260`
  - `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:364`
  - `internal/runplan/conflict.go:2`
  - `internal/websterengine/geometry.go:44` (`Geometry`)
  - `internal/websterengine/state.go:49` (`ScratchDir`)
  - `websterengine/doc.go:824-829`
  - `runlevel_test.go:163` (`seedConflictRecord`)
- **F11:** the loom readers of Webster state outside the webster packages are each covered by a card:
  - `failuretrace/trace.go:332` by card 22.
  - `loomshed/gates.go:254` (`ValidatePlan`) and `loomshed/publishfailure.go:21` (`WebsterRecordNote`) by card 19.
  - `loomcli/renumber.go:59` by card 20.
  - `fabriccli/merge_verbs.go:28` by card 23.
- **F9:** the only callers of the two predicates are `addedNoticeLines` (`decision.go:454`) and `notifyLiveRuns` (`decision.go:482`). Both are now in card 21's Edit list.
- **F16:** `internal/lymdirs/doc.go:9` is the only doc that names the tracked-`_lym` exceptions. Card 3 now edits it.
- **F18:** `reworkPriorWebster` is used only inside loomshed. `ArchivedWebsterDirs` and its only loomcli caller (`wiring.go:932`) are replaced by card 22.
- **F13:** the only site lists and key tables are `PATTERN-gate-slot.md`, `PATTERN-test-speed.md`, `gateslot/doc.go`'s tier plan and `planVerifySites`, and cards 9 and 11 cover all four. Other mentions of "Webster's end gate" (`loomshed/doc.go:109`, `gateslot/doc.go:239`, `PATTERN-gate-slot.md:99`) are comparisons, not lists of sites.

Calls:
1. Grep for `plan-conflict` with webster or scratch wording (F12): my context held only the lines the review named.
2. Grep for production readers of `websterengine.LoadState`, `RunInFlight` and `RunActive` (F11): I needed the full set of readers.
3. Grep for callers of `decisionNoticeInline` and `heldBackReason` (F9): I needed every call site.
4. Grep for `RaddleDirName` mentions and the rework slot-name users (F16, F18): I needed every doc and user.
5. Grep for `publishfailure`/`WebsterRecordNote` in the plan, plus `ValidatePlan`'s state read (F11): I had not read card 19.
6. Grep of card 19's targets (F11): to confirm it edits `ValidatePlan` and `WebsterRecordNote`.
7. Grep of docs that list verify sites or Webster gate keys (F13): I needed the full set of docs. This went one call over the cap of 6.
