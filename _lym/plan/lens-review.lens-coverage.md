```yaml
lens: coverage
outcome: reported
```

Coverage lens: does every decision, acceptance criterion and new branch in the plan have a test row that would fail if it broke?

**Findings**

1. MEDIUM — D1, card 9 — An ended entry whose pair is absent has no row.
   - The record's D1 says the absent-pair entry is judged "owner and ended included".
   - Card 9 tests only an active entry, an owned-elsewhere entry and the no-option error. Card 10's ended row uses a fake reconcile, so a real `ReconcileOrigin` that drops or mis-handles the ended verdict when `PairAbsent` is set fails nothing.
   - Evidence: `_lym/plan/09-reconcile-absent-pair.md` Test rows; `internal/fabricengine#reconcileEnded`, which skips Teardown only when `scope.Run == slug`.

2. MEDIUM — D1, card 9 — The card's wording judges the wrong branch, and the case it gets wrong has no row.
   - The record says "the parent's branch is not judged while the pair is absent". Card 9 says the filter keeps "the prime's and Bolt's targets that every scope keeps".
   - A run scope keeps no prime targets. For a task forked from the prime, the parent branch is the prime's own branch, so keeping prime targets means judging the parent.
   - When the records worktree still stands, `ReadOriginFor` finds the origin record, so the `wanted` filter matches the parent entry unless the absent case excludes it on purpose.
   - The card's row asserts "no branch outcome for the slug or its parent" only for one shape. No row covers a parent pair that is not the prime, or a records worktree that is also gone.
   - Evidence: `internal/fabricengine/originreconcile.go:611-640` (`wanted`); `_lym/plan/09-reconcile-absent-pair.md` Intent bullet 2.

3. MEDIUM — D2, card 12 — A failed `git worktree add` has no row, although the record names it as one of the transient answers.
   - Only an unreachable origin (integration row) and the teardown-lock wait (card 10's `ErrPairPushRunning` row) are reached.
   - If `readoptPermanent` wrongly classed a wrapped `createGitWorktree` failure as permanent, no row would fail.
   - Evidence: D2 "a failed `git worktree add`"; `_lym/plan/12-readopt-failed-stop.md` Tests.

4. MEDIUM — D4, card 6 — The case where origin's weft tip descends from the local tip has no row.
   - Card 6 step 2 skips the branch push there. Step 3 then tags origin's tip as well, since that tip is "neither the local tip nor an ancestor of it".
   - The rows cover origin lacking the tip, a diverged tip, the retry and a refused branch push. None covers origin ahead, so neither the second tag nor the lease on origin's tip (card 7) is reached for it.
   - Evidence: `_lym/plan/06-remove-archive-order.md` steps 2 and 3, and its Test rows.

5. MEDIUM — D3 — The record's "every stop or refusal" is only partly carried.
   - The failed-push record's gone-branch way forward (`internal/fabricengine/failedpush.go:233-236`) names `lym fabric remove` for a task pair whose batten record stands. The plan leaves it unchanged under Shared Decision `d3-sites-outside-fabric`.
   - Fabric cannot name batten, but the follow-up could be added where the notice text reaches a reader outside fabric. The plan carries no card for that and has no `Added after Discussion` record of the drop.
   - Evidence: `00-overview.md` `d3-sites-outside-fabric`; rubric "Fidelity to the answer key".

6. MEDIUM — D5 — The acceptance criterion "the rendered tackler prompt carries the `git ls-remote` rule …" is waived.
   - Shared Decision `d5-no-prose-test` drops the test. The reason given is the `PATTERN-test-speed` ban on testing a stencil's static prose.
   - The waiver comes from a PATTERN conflict, but it is still a criterion the plan says holds without a test.
   - Evidence: `_lym/plan/14-tackler-remote-check.md`; `00-overview.md` `d5-no-prose-test`.

7. LOW — D2 — "dobby plans no further pass for it" is argued from `planPass` and pinned by no row.
   - `TestPlanPass` (`internal/dobbyengine/passes_test.go:17-40`) has a `blocked` row but no `failed` row. Card 12 adds none.

8. LOW — D1 — "A resume of a beam project's run … re-adopts its live pair at the anchor its weft records" is tested only at fabric level, in card 4's beam row.
   - Neither card 10's batten-level integration nor its tier0 rows run a beam project's resume.

9. LOW — Card 10 — Two refusal branches of `readoptGonePair` have no row: a `readopt.Reconcile` error, and a `readopt.Prune` error.
   - The tier0 table lists rows for unreachable, owned, ended, stopped, Add errors and `Gone` false only.

10. LOW — Card 4 — Three branches have no row:
    - The not-live answer for a weft absent both locally and on origin (only the archived case has a row).
    - The adopt-only warp probe "never makes `Add` fetch the task branch from origin" (D8).
    - An adopted weft with no origin record being "written none".

11. LOW — Card 5 — Three branches have no row:
    - `DiscardTaskBranch`'s wait-out refusal.
    - `PushAnchored`'s skip under a held removal (only `PushPairAnchored` has a row).
    - The prime's push taking no teardown lock.

12. LOW — Card 3 — Two branches have no row:
    - A failed drop failing the Add.
    - The weft side's upstream-config unset (row 2 checks the warp's `branch.<warp>.merge` only).

13. LOW — Card 6 and D4 — The "code branch not pushed" and "same pause gate" parts of D4 step 2 have no assertion.

14. LOW — Card 7 — The empty-`leaseSHA` branch has no row in a removal: origin had no weft branch and the branch push failed, so the remote delete must be skipped with nothing reported.

15. LOW — Card 8 — "Any other lock error fills the entry's `Error`" has no row.

16. LOW — Cards 11 and 12 — Three branches have no row:
    - A `SlugRecordsTags` ls-remote or fetch failure.
    - A candidate whose child status is absent or does not decode.
    - `landRestoreWayForward` meeting a `Tags` error.

17. LOW — Card 13 — The `Teardown.Run` wiring is driven only at Run-Shed. The Land wording and the owner-held suppression are reached only through the pure `BattenFollowUp` table, not through `Result.Warning`.

Tool calls made beyond my context:
- `lym quarry resolve` on the tendercli ender glyphs, plus a grep of `TestPlanPass`: card 13 cites the ender glyphs and card 12 relies on dobby's handling of a `failed` status, and neither was in my context. The grep's output came back empty.
- The same resolve and a read of `TestPlanPass`'s rows: the first call's filter dropped the resolve output. The glyphs resolved, and the rows showed no `failed` row.
