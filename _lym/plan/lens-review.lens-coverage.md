```yaml
lens: coverage
outcome: reported
```

Coverage lens: each decision, acceptance criterion and new branch mapped to a card and a planned row. These are the gaps.

**Findings**

1. **BLOCKING** (D2, card 1 `Config.PartsChairTimeout`): The chair's default timeout is computed as twice the raw `PlanTimeoutMin`, but the record says twice the run's *effective* `plan_timeout_min`. The template comment says 0 defers to shuttle's `run_timeout_min` (`internal/loomengine/tapestry.yaml:2`), and `LoadTapestryConfig` accepts 0. With 0, the chair default is 0 and takes the table timeout of 0, so the chair gets the same timeout as every writer. The load check "a set value not above `plan_timeout_min`" compares against 0 too, so it refuses nothing. No row covers `plan_timeout_min: 0`.

2. **MEDIUM** (D7, card 18 `validate-plan --part`): The recipe guard ("no `PartsPlanWrite` row") cannot be reached, and its planned row does not reach it. Only `contracts/recipes/tapestry-recipe.yaml` holds `engine: PlanWrite`, and it gains `PartsPlanWrite` in card 17. On darn, `refuseVerbOffRecipe` refuses `validate-plan` first with the plan-verb text (`internal/loomcli/arm.go:186-198`). So the row "the darn recipe refused" asserts the wrong refusal. Either drop the guard as unreachable, or make the row assert the plan-verb refusal.

3. **MEDIUM** (D4/D6 criteria "with a clean tree"; card 16 `loomCLI.wire` `CommitParts` / `CommitPartsPlan`): No row reaches the real commit seams' pathspecs. Uncovered: the parts dir alone; plan pathspec plus `_lym/parts`; a webster record moved by the rotation. `TestPartsWriteEntry` and `TestPartsPlanWriteEntry` only count calls to fake commit closures. The criterion's "Done commits the plan's pathspec and the parts directory with a clean tree, a webster record moved by the rotation included" is argued, not tested.

4. **MEDIUM** (D7/D5, card 7 `NewPartPlanGate`, `NewPartAccept`): No planned row reaches either one.
   - `NewPartAccept` has two branches: nil when no blocking finding, an error carrying the findings otherwise. It is also the Accept the resume path calls with a zero result.
   - `NewPartPlanGate`'s "parse failure of the part directory is a finding, index error stays an error" has no row.
   - Card 16 only checks that "an Accept" is handed over.

5. **MEDIUM** (card 7 `ValidatePart`): Three branches have no `TestValidatePart` row:
   - an absent parts file (error naming `lym loom goto --to Parts-Write`);
   - a malformed one (error naming `lym loom validate-parts`);
   - a part directory `ParsePlan` cannot read.

6. **MEDIUM** (D8, card 7 `JoinParts`): "Refuses when the parts file is missing" (named in D8) has no `TestJoinParts` row. The rows cover only a missing part overview, a failing part and a passed-through refusal.

7. **MEDIUM** (D8, card 6 `JoinParts`):
   - The export check accepts exports created, re-signed (Edit arrow) or deleted. Only refusal rows are planned. No row shows an export satisfied by a re-sign or a Delete passing, so a check that only honours Create targets passes the tests.
   - The D8 overview rule "each part's `## Shared Decisions` and `## Rename mechanic` carried over" has no row.

8. **MEDIUM** (D10–D12 criterion "the plan-format block names no step", card 12): The card asserts it in prose only. No row checks that the block's text holds no `Step`.

9. **MEDIUM** (D12 criterion "Plan-Write's composed prompt keeps its content"):
   - Card 12's `TestPlanTable` check only looks for subsection headings, two sentences, the gate tiers and the specs dir. A dropped or altered paragraph inside a moved section would still pass.
   - Fidelity: card 12 moves the glyph subsection from Step 2 to Step 3 and adds a new `### Plan-wide rules` heading. Neither is a cross-reference rewrite, which is the only change D12 and its added decision allow.

10. **MEDIUM** (D1 criterion "with it true the run records Plan-Write Skipped after Parts-Plan-Write's Done and reaches Plan-Bouncer"; card 17): `TestPlanSplitRows` checks gate truth values and edges only. No run-level row drives the flag-on sequence and asserts the recorded Skipped entry and arrival at Plan-Bouncer. Only the flag-off sequence is pinned.

11. **LOW** (D1 criterion "a goto onto Plan-Write in a split run runs it"; card 17): The card argues it holds through the generic `TestGoto_StepResumesAtTarget` (`internal/shedengine/goto_test.go:230-233`, an off entry gate bypassed). It plans no split-recipe row.

12. **MEDIUM** (D1, card 20 `segmentWriter`): The new walk ("while the latest entry is Skipped, step to the row whose OnDone names it; stop at a row with no such entry or none before it") is reached only one step deep. No row covers the walk continuing past a Skipped `Parts-Plan-Write`, or ending with no row before it. A bug in the loop's continue or stop branch would pass.

13. **LOW** (card 4 `checkHandleConsistency`): "A `Part.Handles` handle never counts toward `handle-collision`" has no row, such as a part that also declares a handle it was handed.

14. **LOW** (card 15 `NewPartsPlanRotator`): "A failure of the plan rotation returns before the parts directory is touched" has no row in `TestPartsRotators`.

15. **LOW** (card 16 `resolvePlainGateClosure` `parts`): The refusals for `PartsMax` ≤ 0 and for missing `AnchorPath`/`DecisionRecordPath` have no row. Only a valid `parts` row is added to `TestResolveGateSpec_ValidOneEntryPerVocabularyName`.

16. **LOW** (card 11 `writerExit`): No default branch is stated for a writer step end outside the listed cases, such as an empty outcome with no wait error. Its mapping is undefined and no row covers it.

17. **LOW** (cards 13 and 14): Four branches have no row:
    - `PartsTable`: an unreadable depth fragment failing the table.
    - Both tables: a friction-directive render failure logged and left empty.
    - `validatePartsCmd`: a `LoadTapestryConfig` error envelope.

**Tool calls**
- Grepped `TestGoto_StepResumesAtTarget` to confirm it bypasses an entry gate, which my context had not shown.
- Grepped `contracts/recipes/*.yaml` for `engine: PlanWrite` and `engine: PartsPlanWrite` to find which recipes could reach card 18's `--part` guard, which my context had not shown.
