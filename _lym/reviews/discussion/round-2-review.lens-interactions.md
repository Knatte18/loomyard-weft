```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked every pair of decisions that act on the same state and ran each criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (where a loosened check's cause comes from): held. Both accept a trailer on any commit of the review range.
- D2 × D3 (trailer against the stored ruling): held. The ruling key includes the range's trailer lines, so a commit that only adds a trailer gets the file ruled again.
- D2 × D4 (PR-Review's in-place fix commits that nothing checks): held in wording. D2's Bound defers to D4's cap and the gaps D4 names. The cap itself breaks; see findings 1 and 3.
- D3 × D4 (Drift-Recheck after a send-back episode): held. Commits from Code-Burler-Fix and from PR-Review before the send-back are in range and get ruled.
- D3's uncaused Stuck way forward × D1 (a trailer commit, then resume): held. D1 sees the same trailer.
- D4's forced full generation × an amendment over a subject the in-place fixes changed: held. D4 says a send-back file forces a full generation in either case.
- D4's review input list × the Parent-Align fixer report and the amendment note: held.
- D4's fix-here fallback × darn's Describe gate `budget.Code-Review` and Code-Bouncer's `zero_rounds`: held.
- D4's fix-here count × its own send-back clearing and the PR-Review row's re-entry and re-step: broken (finding 1).
- D4's send-back mark × the continue flag on the same record: broken (finding 2).
- D4 × D5 (the threshold against the run_settings override): broken (finding 3).
- D4 × D6 (one findings file and one friction note per send-back): held. `NotePath` picks a free name, and the file is numbered per send-back.
- D4 × `parent_review_asked` and the session id cleared on reroute: held.
- D7 × D1 (figures in the change description against the review checks): held.
- Constraints × decisions (tolerant records for the new record keys, refusal ways forward for the verb and for the uncaused Stuck, rows-not-name for the edge): held.
- Acceptance criteria × D2–D6: held, except the D4 criterion "a new PR-Review episode starts with a fix-here count of zero", which inherits finding 1.

**Findings**

1. **MEDIUM, design, `_lym/discussion/decision-record.md#D4`.** The fix-here count's reset point is undefined, and both readings break D4's own bound.
   - D4 clears the count "when an episode of its segment begins". The PR-Review row has no `segment:` in any of the three recipes, so it has no segment episode (`contracts/recipes/tapestry-recipe.yaml:523-525`, `darn-recipe.yaml:279`, `sampler-recipe.yaml:241`).
   - In shed, a reroute ends only the episode of rows that share the *target's* segment (`internal/shedengine/doc.go:112`). A PR-Review row has none, so its episode ends only at its own Done (`internal/shedengine/run.go:632`).
   - The clean-tree send-back clears the mark, `parent_review_asked` and the session id, but not the count (decision-record.md:110).
   - **Reading 1, the count is kept across a send-back:** the fixes made before the send-back stay counted after Code-Review has re-reviewed the whole range. The first small set on re-entry then sends the run back again, until the reroute cap halts it.
   - **Reading 2, the row clears the count on each `Call` entry:** a transient GitHub or push failure re-steps the row (`prreview.go`, `Call` doc), which zeroes the count, so the "at most `pr_review_small_max` per episode" bound evaporates.
   - Fix: say the send-back resets the count alongside the mark, and that nothing else does; a re-step or resume keeps it. Tie the criterion "a new PR-Review episode starts with a fix-here count of zero" to that event.

2. **MEDIUM, design, `_lym/discussion/decision-record.md#D4`.** The send-back mark and the continue flag can both be set, and D4 never says which wins.
   - On a clean tree at the gated head, today's wake stops the session for `rec.Continue` and `judge` ends the review Done through `endReview`. It never looks at another record field (`internal/landingshed/prreview.go:686-692`, `883-901`).
   - If the agent runs `lym loom pr-continue` after `pr-findings` answered send-back, for example because the operator says continue, the run can land with the send-back never taken.
   - Fix: state that the send-back mark takes precedence over the continue flag in the wake, or that `pr-continue` refuses while a send-back mark is set and names its way forward.

3. **LOW, design, `_lym/discussion/decision-record.md#D2`.** D2's and D4's bounds rest on a cap that a task's `run_settings` can raise without limit.
   - D2's Bound and D4's Bound both lean on "at most `pr_review_small_max`" in-place fixes (decision-record.md:49, 132).
   - D5 lets a task set that value in its `run_settings` with no ceiling and no stated category (decision-record.md:144-146).
   - A large value turns the send-back off without either Bound saying so.
   - Fix: state in D5 what a high value lets through and what bounds it, or classify the key so `run_settings` cannot raise it.

No tool calls made.
