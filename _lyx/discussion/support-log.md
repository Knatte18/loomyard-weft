# Support log — run-reentry

## Interview

Autonomous run: no operator answered; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — what goto resets today, and where the fresh review came from

Read `shedengine/goto.go` and its package doc: a goto entry ends the target segment's bounce episode (`gotoEndsEpisode`), and `Goto` and `Relaunch` write `paused` with no reason.
Read `Bouncer.Call`: at entry a CONVERGED or settled-accept verdict takes the clear (`settledGeneration`), which archives the run directory and re-seeds round 1.
Its Warn log says a commit-seam failure followed by a resume takes this path.
So in `ubuntu-2404-support` the failed commit made every resume re-seed a review, each seed Stuck spent budget, and the goto then reset the count.
The fix in the Bouncer is the main one: a settled generation is finished or re-settled, and only an amended subject gets a new, small generation.

### Turn 2 — amendment budget

The Shed-level episode after a converged Done is already fresh, so the full fresh budget an amendment faced came from the Bouncer's new generation, not from Shed.
Picked: the amendment generation's soft and hard cap come from one allowance key, clamped to the perch's caps.
The PR-Rework exempt skip (`PlanReviewSkippable`) already skips an exempt rework entirely; the allowance covers the non-exempt rest, and darn's Darn-Write after a PR-Gate reject.

### Turn 3 — budget-exhausted way forward

With goto keeping the budget, the budget-exhausted halt's goto way forward stops working.
Picked: a budget-capped entry ends the episode, so a resume of that halt grants one more budget, mirroring the reroute cap; the driver hands that halt back so only the parent grants it.

### Turn 4 — pause reason

`Run` already computes a reason for every paused exit but persists none; `Goto` and `Relaunch` compute none.
The #759 sequence (failed, then paused at Plan-Bouncer after a stuck entry) matches a re-launch with a target.
Picked: a persisted `pause_reason` written by every paused write.

### Turn 5 — friction per stop and the zero-fork round

`reflectHalt` spawns a reflection at every blocked and failed halt, from the step hook, the loop and arm.
Picked: keep the halt note, drop the reflection; Friction-Reflect covers the notes at the end.
`auditClusterRound` fails a shortfall hard after a `done`; the Burler review row already retries a died or timed-out first attempt.
Picked: a fork shortfall joins that retry, and the error names the chair's session-end states.
On the board's shuttle item: the `idle-stalled cause=interrupt` then `done` sequence is benign under `PATTERN-completion-signal`, since the chair's outputs existed; the defect is burler accepting the shortfall as final, which D9 fixes.

### Turn 6 — lens review

Seven forks reviewed the first draft; all seven returned a report.
The writer checked loom's verify-failure target in `landingdeps.go` (`Webster-Burler-Review`) and `segmentBounces` (counts through `Routing.Bounces`) to settle two disputed points.

- interactions BLOCKING, D2 blocks loom's verify retry onto `Webster-Burler-Review`: fixed, D2 admits the row the Publish failure record names.
- interactions MEDIUM, `approveWithoutReview` writes no settle record: fixed, D5 writes one on every approve path and reads it at round 0.
- interactions MEDIUM, a goto entry reads as "latest entry not Done" in D5 case 1: fixed, D5 no longer reads history; the settle record is written before the settle's steps.
- interactions LOW, `pause_reason` ownership against outside writers: fixed, Shed-owned and carried through by outside writers.
- interactions LOW, how the Bouncer treats the reroute's round N+1: fixed, one line under Scope Out.
- interactions LOW, Shed's generic block with an unwired seam: fixed, stated as D6's bound.
- existing-paths BLOCKING, PR-Rework archives the review run directories, so D6 never fires on that route: fixed, every archive keeps the settle record.
- existing-paths MEDIUM, a goto to Plan-Write rewrites the whole plan: fixed, D6's bound says any amendment size gets the allowance.
- existing-paths LOW, `gotoEndsEpisode` also serves the reroute: fixed in D1 and the plan-writer notes.
- existing-paths LOW, arbiter or tackler re-launch refused by D2: fixed, it hands back with the refusal text.
- existing-paths LOW, `bootstrap.go`'s reflection allowance is stale: fixed, D8 retires it.
- bound-breaker BLOCKING, failed commit then goto to the writer approves a changed plan unreviewed: fixed, D5 compares identity in every case.
- bound-breaker MEDIUM, code identity changes at Merge-In and the diff carries parent changes: fixed, the code identity is the task's change against its merge base.
- bound-breaker MEDIUM, the pause reason does not say who paused: fixed, the pause verbs take `--by`; tender pauses write no status (Scope Out).
- bound-breaker MEDIUM, the `Bounces` seam may not see `budget_capped`: rejected, it counts through `Routing.Bounces`, which shares `episodeStuckCount`; a criterion pins it.
- bound-breaker LOW, the D3 rule's position: fixed, ahead of every tackler or arbiter rule.
- bound-breaker LOW, idempotent Approve and Commit claimed, not verified: fixed, stated as a requirement on the seams.
- bound-breaker LOW, D9 is a new error-path branch, and the archive must cover the chair's review: fixed in D9.
- bound-breaker LOW, the PATTERN text's "webster reflection": fixed, removed.
- bound-breaker claim that the verify goto targets a writer row: rejected, loom's target is `Webster-Burler-Review` per `landingdeps.go`.
- walkthrough MEDIUM, reroute and `gotoEndsEpisode`: merged with existing-paths, fixed.
- walkthrough MEDIUM, goto entry in case selection: merged with interactions, fixed.
- walkthrough MEDIUM, zero-allowance approval leaves no record: merged with interactions, fixed.
- walkthrough MEDIUM, code identity changes at Merge-In: merged with bound-breaker, fixed.
- walkthrough LOW, the chair's session states have no path to the error: fixed, D9 says the review result carries them.
- walkthrough LOW, D3 rule order: merged, fixed.
- walkthrough LOW, the exempt skip leaves the old settle record: fixed, `approveWithoutReview` writes the record.
- goal-scope MEDIUM, the board's converged-round budget halt reads as dropped: fixed, D3 states D5 makes it unreachable.
- goal-scope MEDIUM, the pause reason does not say who paused: merged with bound-breaker, fixed.
- goal-scope MEDIUM, existing goto way-forward texts not swept: fixed, a Scope line and a criterion.
- goal-scope MEDIUM, `batten abandon` could delete unreflected notes: fixed, named in Open risks for `run-process-ownership`.
- goal-scope LOW, #759 cause asserted: fixed, an assumption plus a criterion for the re-launch.
- goal-scope LOW, the shuttle item has no finding: fixed, Turn 5.
- goal-scope LOW, what goto still resets: fixed in D1.
- goal-scope LOW, the goal omits the budget halt: fixed in Goal.
- board-fit BLOCKING, D2 against the verify retry: merged with interactions, fixed.
- board-fit MEDIUM, overlap with `orchestrator-survey-ideas`: fixed, a Scope Out line.
- board-fit MEDIUM, D3 for every shed recipe: fixed, the rule lives in `sheddriver`.
- board-fit LOW, `run-process-ownership` inherits no friction session: covered by D8 and the Open risk.
- board-fit LOW, tender pauses record no `pause_reason`: fixed, Scope Out.
- board-fit LOW, batten's resume verb for the budget halt: fixed, D3's way forward is told per recipe.
- leanness LOW, the #759 sentence in D4: fixed, moved to an assumption.
- leanness LOW, the case-4 restatement in Constraints: fixed, cut.
- leanness LOW, auto-mode bullets restate D6 and D7: fixed, shortened to pointers.
- leanness LOW, D6's route list: fixed, only the two PR-Gate routes stay.
- leanness LOW, D3 names verbs: fixed, the way forward is told.
- leanness LOW, the third plan-writer note: fixed, cut.

## Rejected alternatives

- Goto with a `--fresh-budget` flag: keeps the reset one flag away, which the operator called extremely fragile.
- Amendment budget as the remaining budget plus the allowance: an early converge would leave nearly the full budget, which the operator ruled out.
- Raising the perch's budget in a local config layer as the budget-exhausted way forward: PR-Gate's budget is a recipe literal no config raises.
- Nudging the live chair to spawn its forks after a shortfall: the chair had already written its review and ended; a fresh attempt is the existing, tested path.
- Detecting an amendment or an interrupted settle from Shed history (a writer row Done, or the Bouncer's latest outcome): a goto entry and a failed commit followed by a writer goto both misread, and the lens review broke the first draft on exactly that.
- Exempting every automatic goto from D2: only loom's verify retry needs a settled review row, and its failure record names the one row.
- A code-segment identity from the raw tree: a Merge-In alone would read as an amendment, and the diff would carry the parent's changes.
- Persisting the pause reason in the existing `error` field: `error` reads as a failure in every consumer, and paused is no failure.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Should goto still reset the segment's budget? — No; D1.
- Q2 (auto-pick): How is a goto onto a settled review row handled? — Refused through a told seam; D2.
- Q3 (auto-pick): What replaces goto as the budget-exhausted way forward? — A parent-decided resume of a budget-capped halt; D3.
- Q4 (auto-pick): Where is the pause reason kept? — A new `pause_reason` status field; D4.
- Q5 (auto-pick): How does the Bouncer tell an interrupted settle from an amendment? — A settle record with the subject identity; D5.
- Q6 (auto-pick): What budget does an amendment get? — The allowance alone, clamped to the perch caps; D6, D7.
- Q7 (auto-pick): Per-perch or run-wide allowance key? — Run-wide only; D7.
- Q8 (auto-pick): Does the tackler or arbiter chain change? — No; only the friction session goes; D8.
- Q9 (auto-pick): Retry or nudge a zero-fork round? — Retry as a fresh attempt; D9.
- Q10 (auto-pick): Does the Webster reroute edge change? — No; it is outside the board entry.
- Q11 (auto-pick): Does loom's verify retry onto a settled review row stay? — Yes, admitted by D2 for the row the failure record names.
- Q12 (auto-pick): How does a pause say who asked? — A `--by` label on the pause verbs; D4.
- Q13 (auto-pick): What is a code segment's subject identity? — The task's change against its merge base with the parent; D5.
