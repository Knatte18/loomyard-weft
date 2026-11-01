```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's proposals the fixer carried out, F1–F8.

**Findings**

1. **MEDIUM — F6, the renamed hand-back fakes now invert production's rule.**
   - Changed text: `internal/sheddriver/drive_test.go` (`websterCapStatus`, and the predicate `producer == "Webster"`), `internal/shedverbs/status_test.go` (`{"BlockedFinalizeRerouteCap", "Finalize", …, false}`) and `internal/battenshed/innerrun_halted_test.go` (`{name: "FinalizeHaltIsReSteppedAsToday", producer: "Finalize", wantSpawns: 1}`).
   - Summary: the tests now say a Webster reroute-cap halt is handed back and a Finalize halt is not, the reverse of the documented behaviour.
   - Evidence: `docs/overview.md` says "A halt at `Parent-Align` or `Finalize` (a spent or over-cap fixer, fix reviews out of rounds) is handed back to the orch through every driver". The drive_test case name still reads "a status moved to a blocked landing halt hands back with no tackler…", and the battenshed test comment still says "landing-halt row". Both now describe a Webster status. The reviewer's "such as Webster" proposal turns a hand-back example into a counter-example for the row that is actually handed back.
   - Better change: keep Finalize as the handed-back row and give it a halt reason `HandsBack` can still produce, such as a spent cap or `LandingStopPrefix`. Alternatively keep Webster, but then fix the case names and comments.

2. **LOW — F4, the new head-read-failure subtest does not assert what its name says.**
   - Changed text: the subtest "a task head that cannot be read is an error and the entry stays unaccepted" in `fixaccept_test.go`.
   - Evidence: it checks the error and that the description is untouched. It never reads the attempt record to confirm `AcceptedHead == ""`.

3. **LOW — F5, the new assertion sits in a test named and documented for Webster.**
   - Changed text: the `p.Name == loomshed.NameFinalize` check in `TestRecipe_WebsterReroutesToPlanReview` (`internal/loomrecipe/recipe_test.go`).
   - Evidence:
     - The review named `tapestryrecipe_test.go`, which does not exist.
     - The check now lives in a test whose name says only Webster. Searching for a Finalize no-reroute pin by test name will not find it.
     - If the tapestry recipe lost its Finalize row, the loop would pass without checking anything.

4. **LOW — F1, an acceptance is lost when a resume moves the task HEAD.**
   - Changed text: `rec.AcceptedHead == head` in `judgeFixerChange` and the reset after it.
   - Case: the operator accepts at HEAD `h`, then `lym loom resume` re-merges a parent that has moved. The merge commit changes HEAD, so `AcceptedHead != head`, the acceptance is cleared, and the fix reviews run again over `Baseline..head`, which may now include parent changes.
   - Context: the existing `Reviewed == head` check has the same property, so this is parity, not a new regression. The reviewer's alternative of excluding the description commit would not fix it either. The effect is more review rounds, not an unreviewed landing.
   - Also noted: the JSON key changed from `accepted` to `accepted_head`. A record written by the old build that is already accepted silently loses its acceptance, because unknown keys are tolerated. The result is a re-review, which is safe.

**No findings:** F2, F3, F7 and F8.
- F2: the rows fail if the `FixReview` clear or the fixer spawn is removed.
- F3 and F7: the wording agrees with the overview line that describes `Parent-Align` and `Finalize` halts.
- F8: the dropped second call is covered by the seeded spent-cap row, which has the same head and commit range.

Extra tool calls: none; my context already held everything this lens needed.
