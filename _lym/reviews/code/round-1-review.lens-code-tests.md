```yaml
lens: code-tests
outcome: reported
```

Scope: the code-tests lens. For each behaviour the diff adds or changes, I named the test row that would fail if it broke.

**Per-behaviour list**

- `InClosedScope` row span, `roundStarted`, Finalize exclusion, empty or unknown row, no-segment recipe, unknown recipe: pinned by `TestInClosedScope` (scopeline_test.go:12–50). Sampler has no row, but it is the same row shape as darn.
- `CodeReviewRunSubdir`: pinned by `TestCodeReviewRunSubdir`.
- `scopeClosedAt` reading the real Code-Review run directory (missing dir reads as 0 rounds, a round present closes the scope at any row, other stat or LatestRound errors): **none**. The verb tests replace `scopeClosed` with a fake.
- `scopeClosedRefusal` text, and its wiring into `decisionVerb` and `decisionReplaceVerb` for parent, operator and read failure, with nothing written or posted: pinned by `TestDecisionCalls_RefusedOnceTheScopeCloses`. The `cmp.Or(currentRow, "no row")` branch has none.
- `RecordInScopeContinue` success, other cause, non-decision id, unknown id, round not escalated: pinned by `TestRecordInScopeContinue`. Three paths have none: round 0, an existing decision file (`ErrCirclingDecided`), and an unreadable review.
- `InScopeFindings` round-trip: pinned by `TestRecordInScopeContinue` and the new `TestBuildRoundContext` row. A malformed file has none.
- `BuildRoundContext` marks only held findings that the ruling names: pinned by `InScopeRulingMarksOnlyTheHeldFindingsItNames`, plus the no-ruling assertion in the existing row.
- `circlingVerb` `in_scope` envelope and record pass-through: pinned by the `TestCirclingVerb_Records` row "continue ruling findings in scope".
- `circlingVerb` refusals for `ErrInScopeCause` and `ErrInScopeFinding`: pinned by two `TestCirclingVerb_Refusals` rows.
- `preRun` record closure that routes a non-empty `--in-scope` to `RecordInScopeContinue` and returns cause decision, and the `--in-scope` flag registration: **none**.
- `roundCheck.priorEntry`, where an in-scope finding resolves without a quoted title: pinned by `InScopeFindingResolvedWithoutAQuotedTitlePasses`. The outdated status is still refused: `InScopeFindingOutdatedIsRefused`.
- `carriedFinding`: an in-scope finding carried as decision is refused (`InScopeFindingCarriedAsADecisionIsRefused`), carried as fixer work it passes (`…CarriedAsFixerWorkPasses`), and a held finding not in scope keeps its class (`HeldFindingCarriedWithAnotherClassIsRefused`). The last one guards the `!prior.InScope` condition, which `CarriedClassDiffers` does not reach for a held finding, so it is not redundant.
- `writePriorFinding` `inScopeMark` case: **none**. prompt_test.go:529 asserts only `heldByParentMark`.
- `noticeWayForward`, `escalationReason`, `answerRoute` and `decideVerbs` on a closed scope below the hard cap: pinned by the `TestBouncer_DecisionEscalation` subtest "AClosedScope…".
- The closed-scope branches at the hard cap (`noticeWayForward` second case, `escalationReason` with `atHardCap`): **none**.
- Open-scope `answerRoute`, which reproduces the old fragment line: pinned by the existing decision-escalation subtests that assert the brief.
- Markers set of the decision fragment: pinned by `TestStencils_TopLevelMarkers`.
- `shedrecipe` `scope_closed` parse and accept, and its pass-through to `BouncerConfig.ScopeClosed`: **none**. The shipped recipes only parse it implicitly. Which shipped rows set `scope_closed` is pinned by nothing: `decisionswitches_test.go` pins `escalate_decisions` and the other switches, but not this one.
- `gateslot` negative budget refused: pinned by the `TestLoadConfig` row. TiersValue budget line: pinned by the `TestConfig_TierUse` addition. Zero means no line: pinned by the existing `twoTier` assertion. Template default 60: pinned by `templateDefaults`. Category: pinned by `TestModules_EveryTemplateKeyClassified`.
- Loosened checks: none found. The edited `circling_test` record assertion is equivalent or stricter. The `bouncer_escalation_test` and fixture edits only change a signature or add a key. The markers row follows the new marker set.

**Findings**

1. MEDIUM, design. `internal/loomcli/decision.go` `scopeClosedAt` (diff 014, lines added around decision.go:944–962).
   - Summary: the half of the scope line that reads a real run directory has no test. That half is "a Code-Review run directory holding a round closes the scope whatever row the run stands at", plus the missing-dir and stat-error paths. `TestDecisionCalls_RefusedOnceTheScopeCloses` fakes `scopeClosed`, and `TestInClosedScope` takes `roundStarted` as a bool. So the goto-after-a-round case that the summary claims is covered is pinned only at the bool.
   - Evidence: decisionreplace_test.go `f.closedRows` fake (diff 014 test hunk); decision_test.go:481 fake `scopeClosed`.
   - Fix: add a tier0 table over `scopeClosedAt` with a `t.TempDir()` reviews dir. Rows: no `code` dir, `code` with `round-1-review.md` at Darn-Write (closed), empty `code` at Darn-Write (open), and arras (no line).
2. MEDIUM, design. `internal/shedrecipe/entries_bouncer.go:88` (`scope_closed` parse) and the three recipe yamls.
   - Summary: nothing tests that `scope_closed` reaches `BouncerConfig.ScopeClosed`, or that the Code-Bouncer rows of darn, tapestry and sampler set it. Dropping the key from a recipe, or the pass-through line, fails no test, and the closed-scope wording would silently revert to "record with decision add".
   - Evidence: `entries_bouncer_test.go:1428–1429` cover only `escalate_decisions`; `decisionswitches_test.go:28–70` pins `escalate_decisions` and its siblings, not `scope_closed`.
   - Fix: add `scope_closed` to the `decisionswitches_test` table, and add an `entries_bouncer_test` row whose config sets it and whose escalation carries the closed-scope route.
3. LOW, design. `internal/loomcli/circling.go:225–231` (`preRun` record closure) and the `--in-scope` flag registration at circling.go:796.
   - Summary: the real routing from a non-empty `--in-scope` to `RecordInScopeContinue` with cause decision is untested. Every circling test injects `circlingFake.record`.
   - Evidence: circling_test.go:56–65.
   - Fix: cover the wiring at the cobra command level with a run dir, or note why the wiring stays unpinned.
4. LOW, design. `internal/burlerengine/prompt.go:557` (`writePriorFinding`, `case prior.InScope`).
   - Summary: no prompt test asserts that `inScopeMark` replaces `heldByParentMark` for an in-scope prior. The reviewer's only signal for the class change is unpinned.
   - Evidence: prompt_test.go:529 asserts only `heldByParentMark`.
   - Fix: add a prompt row with `PriorFinding{Held: true, InScope: true}` that wants `inScopeMark` and not `heldByParentMark`.
5. LOW, design. `internal/shedadapters/bouncer.go:1399` (closed scope at the hard cap in `noticeWayForward`) and `escalationReason` at the hard cap with `ScopeClosed`.
   - Summary: the hard-cap closed-scope wording, a board note only with accept alone, has no test. The new subtest runs only below the cap.
   - Evidence: bouncer_decision_test.go:290 (no `HardCap`) against :621–648 (hard cap without `ScopeClosed`).
   - Fix: add a closed-scope case to the hard-cap subtest.
6. LOW, design. `internal/shedadapters/circling.go` `RecordInScopeContinue` and `InScopeFindings` (diff 017).
   - Summary: four refusal or error paths are untested: an existing decision file (`ErrCirclingDecided`, which must leave the file untouched), round 0, an unreadable review, and a malformed decision file in `InScopeFindings`. The first is a stated contract in the doc comment.
   - Evidence: `TestRecordInScopeContinue` rows (circling_test.go diff), which always write the review and never pre-create a decision.
   - Fix: add rows for a pre-existing decision file and for no review file to `TestRecordInScopeContinue`.

Tool calls:
1. grep of the test files for `scopeClosedAt`, `inScopeMark`, `scope_closed` and the class-change rows, and of the hard-cap subtests. My context had the diff's tests but not the existing tests around them.
2. Read of the `scope_closed` and `escalate_decisions` rows in `entries_bouncer_test`, `decisionswitches_test` and `roundcheck_test` 160–170, to check whether the recipe key is pinned and whether the new held-class row duplicates `CarriedClassDiffers`.
