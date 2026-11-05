---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/22-driftcli-retire-verbs.md:14
    summary: card 22 deletes the constant compareLockRetry although checkDeps in internal/driftcli/wiring.go still reads it after every card, so webster's begin-batch refuses the plan
    fix: In _lym/plan/22-driftcli-retire-verbs.md remove the Delete bullet for internal/driftcli#compareLockRetry and change nothing else in the card; do not move the delete to a card after card 26, since the constant stays live. The compare closure of worktree.checkDeps (internal/driftcli/wiring.go:247, request.LockRetry = compareLockRetry) is the check verb's Compare seam under diff command, which cards 14 and 26 keep, and card 22's compareCmd at internal/driftcli/cli.go:316 is the only reader that goes. Checked with lym loom validate-plan --overlay; the baseline with {} reports caller-uncovered/25 and delete-before-reference/22 as blocking, and the overlay with this one Delete line removed reports only the caller-uncovered/25 baseline finding, so the fix adds no blocking finding.
---

### [BLOCKING] Card 22 deletes a constant the check verb still reads

**Location:** `_lym/plan/22-driftcli-retire-verbs.md:14`, the `internal/driftcli#compareLockRetry` bullet under `**Delete:**`.

**Issue:** `.lym/webster/plan-conflict.md` lists this as a blocking finding of begin-batch 20 (`delete-before-reference/22-driftcli-retire-verbs`), so it is a finding of this round.
The gate's own suggestion, moving the delete to a card after card 26, does not work.
In the landed code `compareLockRetry` (`internal/driftcli/cli.go:36`) has two readers: `compareCmd` at `internal/driftcli/cli.go:316`, which card 22 deletes, and the compare closure of `worktree.checkDeps` at `internal/driftcli/wiring.go:247`, which card 14 added.
That closure is `lym drift check`'s `Compare` seam under `diff: command`, and no card removes it: card 26 edits `checkDeps` only to stop passing a skip list, and card 22 keeps `check` in the command tree.
Any card that deletes the constant therefore breaks `driftcli`'s build, so the constant is not to be deleted at all.
Its doc comment, which says it bounds how long a compare retries a compare marker a reader's probe holds, stays true for the check verb's compare.

## Rejected

- Focus 1, each refusal naming `lym drift defer`, has a row that reaches it, and no refusal outlives the card that replaces it.
  `TestDriftExpect`'s `TheCaptureNeverCounted` row seeds a scope record with `awaited_at` empty and `TheBaselineDirectoryIsMissing` removes the baseline directory, each asserting the `DecisionHaltPrefix` then `DeferHaltPrefix` opening, with no agent spawned and no seal commit; `expect.go` reaches that Stuck before `CheckBaseline`, so no upstream check preempts it.
  `TestDriftVerify`'s `ACaptureThatNeverCounted`, `AMissingBaselineWithNoCaptureRecord`, `AnEditedBaseline` and `AFailedCurrentCapture` rows reach card 16's Stucks.
  Card 17 replaced card 16's interim `command`-mode Stuck and its row together: the landed `verify.go` branches on `DiffCommand` into the command pass, and `verify_test.go` has no interim row left.
  Drift-Baseline's capture Stucks name `lym drift defer` behind `CaptureHaltPrefix`, and `baseline_test.go` asserts `CaptureHaltPrefix + DeferHaltPrefix` on each.
- Focus 2, the row constants: `darnWriteRow` and `websterRow` are declared once, in `internal/driftengine/verify.go`, by card 16, which is also the first card that compares `Deps.Rows` against them; card 19 is later and only reads them, and creates `recheckRow` itself.
  No production file in `internal/driftengine` or `internal/driftcli` repeats a row-name literal.
  The literals `"Drift-Verify"` and `"Drift-Recheck"` in the `darnRows` and `tapestryRows` test fixtures of `verify_test.go` are test-only, and card 16 could not name `recheckRow` before card 19 created it, so no finding.
- Focus 3, the parser refusals: every refusal card 5 and card 6 prescribe has a row in the landed `TestParseExpectation` and `TestParseJudgement`, with no new test function.
  Card 5's row list omits the refusal of an `n` the baseline holds fewer occurrences of; the landed test carries it as `OccurrenceBeyondBaseline`, and card 5 is done and frozen, so there is nothing to add.
  Card 6's "a missing case ruling" row is covered by `RulingOutsideCommandSet` and `EmptyListAfterFailingCompare`, since `JudgementWant` carries no case list for the parser to miss one against.
- Focus 4, card 15's exempt paths: the Bound states what an edit under the run directory, the friction directory and its archive siblings lets through (it goes unseen, and can mislead loom's run state, settling or a stop report) and what bounds it (those paths hold no code, no expectation, and neither `scope.json` nor `judged.json`, which are the records D9 forbids the judge to edit).
  One imprecision, no finding: the run directory also holds `seed.json`, from whose `recipe` `loomcli` derives `Deps.Rows` (`internal/loomcli/arm.go:132`, `wiring.go:949`), so containment reads it as well as the status row; a valid recipe name that still carries the Drift rows moves no containment verdict, and card 15 is frozen.
- Round 3's deferred item, that `lym drift defer` named by Drift-Verify Stucks in cards 16 to 18 was admitted at Drift-Verify only by card 19, is moot: cards 16 to 19 are all done, and card 19's `Defer` now admits the rows D12 names, with `TestDeferWaive` rows for them.
- `caller-uncovered/25-driftengine-retire-acceptance` (card 25 deletes `internal/driftengine#testFiles`, which `internal/driftengine/check.go` still references at line 274) is blocking in the same baseline run, but this round's gate enforces that check, so it is not raised as a finding.
  The fixer meets it at the gate; the constraint on its fix is that `testFiles` is live in `Check`, so card 25 cannot delete it unless an earlier card moves that reference.
