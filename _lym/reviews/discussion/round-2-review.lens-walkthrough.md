```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. Each decision run end to end on one concrete case.

**Decisions walked**
- D1, a test loosened at the review base with a trailer on a later commit of the range: the rubric reads the range's commit messages, so the trailer clears the finding. Purpose met.
- D2, a Go-written commit for a role whose handoff carries one `Loosened-Check:` line: Go appends it, and D1 and D3 read it from the range. Purpose met.
- D3, a widened limit in an existing test with no trailer, under tapestry: Drift-Verify spawns the judge, the ruling is `uncaused`, and the row halts Stuck for the parent. The parent is then handed "commit a trailer, then resume" with no session in the task worktree to make that commit. Purpose not met.
- D3, Drift-Recheck over an unchanged diff and unchanged trailers: the stored ruling is reused and no judge spawns. Purpose met.
- D4, first entry: the agent fixes 3 findings in place (count 3). A later set of 1 brings the total to 4, more than 3, so the run is sent back. The wake stops the session, clears the mark, `parent_review_asked` and the session id, and reroutes. Code-Bouncer runs a full generation, then Drift-Recheck, Describe, and PR-Review again. On that re-entry the count's reset point is undefined, so it either stays at 3 and the next one-finding set is sent back again, or the count resets on any row re-entry. Purpose not met.
- D4, re-entered PR-Review session: the strand is not live after the stop and the session id is cleared, so `ensureSession` starts a new session. `askParentReview` asks again because the flag was cleared. Purpose met.
- D5, a task with `run_settings` `{loom: {pr_review_small_max: 10}}`: the key's config category is unstated, which decides whether the local layer admits it. Purpose unsettled.
- D6, two send-backs: `friction.NotePath` gives each a fresh name, and the findings files are `pr-findings-1.md` and `pr-findings-2.md`. Purpose met.
- D7, Describe after the perch: the claim rule applies to the final description. Purpose met.

**Findings**

1. BLOCKING, design, `_lym/discussion/decision-record.md#D4`
   - Summary: D4's fix-here count is cleared "when an episode of its segment begins", but no PR-Review row has a segment, and D4 never says what resets the count across a send-back or a re-step.
   - Why it matters:
     - The PR-Review row declares no `segment:` in any of the three recipes, so shed's episode for it runs from its own last Done. PR-Review never reaches Done between send-backs.
     - The send-back clears the mark, `parent_review_asked` and the session id, but not the count.
     - So after a re-entry the count keeps the earlier episode's value. The first sub-threshold set then sends the run back again, until `rerouteCap` halts it.
     - If "episode" is read as any `Call` entry instead, a driver re-step or crash-resume zeroes the count. That lifts the cap the D4 Bound promises.
   - Evidence:
     - The PR-Review rows in `contracts/recipes/tapestry-recipe.yaml:523-525` declare engine and `on_done`, no segment.
     - `internal/shedengine/run.go:631-703`: the episode is counted from the row's own Done when the row has no segment.
     - `internal/landingshed/prreview.go`, `Call` (around line 71): the row is re-entered on every re-step.
   - Fix: in D4, state when the count resets, for example at the send-back itself together with the other cleared fields. State that a re-step or crash re-entry of the row keeps it. Reword the "new PR-Review episode" criterion to match.

2. MEDIUM, design, `_lym/discussion/decision-record.md#D3`
   - Summary: the way forward for an `uncaused` ruling is "a commit carrying the trailer, or one reverting the loosening, then `lym loom resume`". That Stuck halts for the parent, and nothing in the run makes such a commit in the task worktree.
   - Why it matters: every existing Drift Stuck routes the code fix through `lym loom goto --to <writer row>` and then `lym loom resume`. Without that, the parent has to commit in the task worktree itself, and the parent directive forbids committing in another worktree. The D3 criterion added for F1 asserts this same path.
   - Evidence: `internal/driftengine/doc.go:586-588` and `internal/driftengine/verify.go:331` (`"lym loom goto --to " + row + " then lym loom resume"`).
   - Fix: give the `uncaused` Stuck the existing writer-row goto as its way forward, with the trailer or revert done in that row. Reword the D3 Stuck criterion and the trailer-only re-rule criterion to match.

3. MEDIUM, design, `_lym/discussion/decision-record.md#D5`
   - Summary: D5 lets a task set `pr_review_small_max` in `run_settings` but never states the key's config category.
   - Why it matters: a large value turns off D4's send-back, which hollows a review gate. `PATTERN-config-layers` classifies such a key as Guard, and a Guard key is closed to the local layer that `run_settings` writes. So either `PrepareRunSettings` refuses the task's setting, or classifying it None leaves the guard switchable per task with no stated bound.
   - Evidence: `pattern/PATTERN-config-layers.md` ("Guard: … a review or an approval gate"), and the category map in `internal/loomengine/config.go:640-680`.
   - Fix: in D5, state the key's category and its consequence: Guard and not settable per task, or None with the bound on how far a task may raise it.

Tool calls:
- grep of the driftengine and loomshed ways forward, plus the PR-Review segment: the context had not shown how the existing Drift Stuck ways forward are phrased.
