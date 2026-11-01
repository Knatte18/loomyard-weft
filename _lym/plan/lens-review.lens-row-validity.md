```yaml
lens: row-validity
outcome: reported
```

Task: row-validity review of the plan's planned test rows.

1. **BLOCKING**, Card 7 (`TestValidatePart`, `TestJoinParts` in loomshed): the card puts both tests "over a fake index answering no findings, as `TestNewPlanGate` builds". That premise is false: `TestNewPlanGate` uses the real `planglyph.NewIndex(fabricengine.NewReferenceRule())` (`internal/loomshed/gates_test.go:397`, `:429`, `:995`, `:1018`).
   - `handle-dangling` and `part-spot-target` come from `planparser` checks that only `index.ValidateFormat` runs (via `planglyph.validateFormat`, `internal/planglyph/planglyph.go:34-45`). A fake index that answers no findings never produces them.
   - So these rows cannot pass: "a dangling handle no export covers reported" and "a card targeting a spot reported as `part-spot-target`" in `TestValidatePart`, and "a part failing its check refused with its findings" in loomshed `TestJoinParts`.
   - Fix: run both tests over the real index, as `TestNewPlanGate` does.

2. **MEDIUM**, Card 5 (`TestValidateFormat_PartScope`): the `caller-uncovered` row cannot reach `planGatePass` if built like `TestResolvePass_ForthcomingCreateTargetExcludedFromStatus`.
   - That test goes through `forthcomingFindings(t, card1, card2, forthcoming bool)` (`internal/planglyph/planglyph_test.go:380-390`). It is the dispatch form with a forthcoming set, and `ValidateDispatch` leaves out `planGatePass`'s findings (`internal/planglyph/planglyph.go`, ValidateDispatch doc).
   - So "with `Part` nil gives `caller-uncovered`" never holds there.
   - Fix: make the `caller-uncovered` half a row of `internal/planglyph#TestGlyphChain_CallerCoverage` (`internal/planglyph/glyphchain_test.go:766`, over the callees fixture) and list that test on the card.

3. **MEDIUM**, Card 18 (`TestValidatePlanCmd_Part`, row "the darn recipe refused"): this row reaches `refuseVerbOffRecipe`'s existing plan-verb refusal, not the new `--part` refusal.
   - `validate-plan` is in `planVerbs`, which `refuseVerbOffRecipe` refuses on any recipe without a `PlanWrite` row (`internal/loomcli/arm.go:182-198`).
   - Only `contracts/recipes/tapestry-recipe.yaml:227` holds `engine: PlanWrite`, and after Card 17 tapestry also holds `PartsPlanWrite`.
   - So no recipe reaches "`--part` on a recipe with no `PartsPlanWrite` row". The branch is ruled out upstream: drop it, or write it as a fail-loud wiring guard with no reaching row.

4. **MEDIUM**, Card 10 (`TestEngine_Run_WriterEndDecidesTheJoin`): the row "a writer whose gate fails while the chair's end arrives first" has no deterministic sequence.
   - `join` selects between `chairDone` and `ender.signal` (`internal/seatengine/engine.go`, `join`). Releasing both fake handles lets either case win.
   - The card names no ordering step or helper: for example, release the writer, wait until the step ender has fired (`stepEnder.fired`), then release the chair and await. Without one the row is flaky or tests the other order.

5. **LOW**, Test economy, Cards 9 and 10 (`TestEngine_Run_WriterSeat`, `TestEngine_Resume_Writers`): these repeat existing coverage. `internal/seatengine#TestEngine_Run_RequiredAdvisorEndsTheStep` already has one row per end: died, timeout, wait error, failed gate, accept error, failed start, and resume did not find (`internal/seatengine/engine_test.go:503-511`).
   - The writer ending rows, and the resume rows for a writer not found with missing outputs or a failing Accept, should be a writer variant of that table.
   - Only behaviour that test does not reach needs its own test: the done line, no up line, `Writer`/`Outputs` on the result, and the resume line with outputs present.

6. **LOW**, Card 10 (`TestEngine_Resume_Writers`): the row "a writer that had been asked a fix included" adds nothing. No Go-visible state records a fix request, so it drives the same branch as the plain "writer not found, outputs exist" row and cannot fail apart from it. Drop it.

7. **LOW**, Card 14 (`TestPartsPlanTable`): "an Accept that is the told one" cannot be checked by comparing values, since Go func values are not comparable. The row needs a stated observation, such as a told hook recording the part it was built for when called.

8. **LOW**, Card 2 (`TestDecisionIDs`): the card puts the test in a new `decisionids_test.go` "beside the package's other tests". The package's `CheckDecisionIDs` test lives in `validate_test.go` (`internal/discussionparser/validate_test.go:496`, `TestCheckDecisionIDs`), so the file named does not match the existing layout.

Calls made:
- `grep` of `TestNewPlanGate` in `internal/loomshed/gates_test.go`: to see which index its rows use, which my context did not show.
- `grep` for `TestGoto_StepResumesAtTarget`, `TestEntryNumbers`, `TestResolvePass_ForthcomingCreateTargetExcludedFromStatus` and `TestCheckDecisionIDs`, plus the planglyph test tiers: to confirm the tests the cards name exist and where.
- `grep` of loomrecipe `testEnv` and the recipes holding `engine: PlanWrite`: to check the sequence env and whether `--part`'s recipe refusal is reachable.
- `sed`/`grep` of `planglyph_test.go:380-400`, the `caller-uncovered` tests, and the `TestEngine_Run_RequiredAdvisorEndsTheStep` rows: to check card 5's row home and cards 9 and 10's overlap with existing rows.
