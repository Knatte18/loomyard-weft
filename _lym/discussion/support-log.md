# Support log — wxb-plan-without

## Interview

### Turn 1 — exploration (autonomous mode, no operator)

Four explorer subagents answered:

- `lym beam start` never starts a pawl run; the pawl recipe's "bring up the beam's dobby and orch" step calls `orchcli.BeamUp`, after the pawl run's `lym loom start` already added the status strand in the overview window.
  So the order is fixed by pawl itself, not a race inside beam start.
- `ensureRunsStrand` is called only from `beamUp`, and on finding a loom status strand with no runs strand it calls `ReplaceStrand` on it with the runs spec.
- `statusStrandAddSpec` already gives the status strand its own window with `KeepViewer` when a runs strand exists; reed names an own window by the strand's role, or by `Display.Window` when the strand sits in a group.
- `lym batten overview` reads no self run today; `shedrun.ReadSeed`, the status file and `overviewStage` give everything a pawl line needs.
- No path removes the loom status strand.

### Turn 2 — design, auto-picked

Picked the move-in-`ensureRunsStrand` fix over reordering, one shared spec builder, the recipe as the window name and the overview line. See the question ledger.

### Turn 3 — lens review

All six forks returned a report.

- interactions 1 (concurrent loom start adds the role first): fixed; D1 counts a role-held refusal as done, and the loom-start side race is an open risk.
  Round 1 replaced the refusal with an `AddStrandUnless` skip on both sides.
- interactions 2 (failed re-add not retried, warning names the wrong verb): fixed; the warning names `lym loom start`, and the runs sidecar stays written.
- interactions 3 / walkthrough 1 (builder fed the pre-replace strand list): fixed; D2's builder takes the placement as input and D1 asks for beside-runs.
- existing-paths 1 / bound-breaker 1 / walkthrough 3 ("halted" and "finished" are not shed states): fixed; done is `shedengine.StateDone`, pinned to `runStatusUnfinished`, and every other state shows.
- existing-paths 2 (seed and sidecar of which worktree): fixed; D1 names the session's own worktree, which `wireCLI` already sets as the orch's location.
- existing-paths 3 (re-add for a done run): fixed; D1 re-adds only for a self run not done.
- existing-paths 4 / bound-breaker 2 / walkthrough 2 (overview names a window that may not exist; `window_share_max` 0): fixed; D3 reads the window from reed, and the share rule is a pinned Constraint.
- walkthrough 2, side point (KeepViewer on a new shared window): fixed; checked `internal/reedengine/spawn.go`, pinned as a Constraint.
- walkthrough 4 (both build identities must match): rejected; a plan-time check, not a design gap.
- bound-breaker 3 (hand loom start races beam start, "always" overclaims): fixed as an open risk, since the race predates this task.
- bound-breaker 4 / leanness 1 (binary-mismatch "risk" is no risk): fixed; cut.
- board-scope 1 (parent commit `7c23591` lands a different design): fixed as an open risk for the run's parent, class decision; round 2 cut that risk once the added entries settled it.
- board-scope 2 (reorder left out): rejected; the fork itself says no fix is needed.
- leanness 2 (Scope Out reorder bullet repeats a rejected alternative): rejected; the entry offers the reorder as an option, so the plan writer must see it is not taken.
- leanness 3 (never-fail stated twice): fixed; D1 keeps the new add's failure, the Constraint the general rule.
- leanness 4 ("no reed change" three times): fixed; D2's clause cut.
- leanness 5 (assumptions repeat D3): fixed for the halted bullet, now pinned in D3; rejected for the "may" bullet, which D3 does not state.
- leanness 6 (Goal retells the mechanism): fixed; the Goal keeps one sentence of cause.

## Rejected alternatives

- Reorder so the runs strand comes up before the pawl run's loom start: the pawl run's own row brings the beam up, so the loom start always comes first;
  moving the runs strand into the loom bootstrap would have loom know it runs in a beam and duplicate `ensureRunsStrand`'s orch wiring.
- Replace the status strand with an own-window status spec first, then add the runs strand: same end state, but it gives up `ReplaceStrand`'s slot keeping for the overview strand, which the current code relies on.
- Name the window through a reed `Title` on own-window strands: needs a reed change, where `Display.Window` already names a window.
- Name the window by a fixed `pawl` literal: hard-codes a recipe name against `PATTERN-no-hard-coded-choice`.
- Have `lym beam start` restore a missing checklist whenever a loom run lives: covers only an operator-closed strand, which a loom start or resume already restores.
- Have D3 recompute the window name from the recipe: wrong when `window_share_max` is 0, the seed was unreadable at add time, or the strand predates the change.

## Review rounds

### Round 1

- F1: fixed; D3 derives the window from the strand's group and reed's shared record, with a `window_share_max` 0 criterion.
- F2, F3: fixed; D1's re-add and the loom bootstrap's add go through `AddStrandUnless`, a skip counting as done.
- F4: fixed; the re-add runs whatever the runs sidecar write returns, and a failed replace names `lym loom start`.
- F5: held for the run's parent, as Q6.
- F6: fixed; the overview's lock Constraint names reed's shared read lock.
- F7, F8, F9: fixed.

### Round 2

- F10: fixed; the Open risks bullet on commit `7c23591` is cut, and Q6 is marked settled.

## Question ledger

- Q1 (auto-pick): where to fix — in `ensureRunsStrand`, which re-adds the checklist right after taking its window. Rationale: the one place that knows it displaced it, and it covers both orders.
- Q2 (auto-pick): window name — the run's recipe from its seed, through `Display.Window`; fallback to the role when the seed is unreadable. Rationale: meets "named for pawl" with no reed change and no literal.
- Q3 (auto-pick): build the overview line the entry marks optional. Rationale: it points the operator at the window and reuses readers the overview has.
- Q4 (auto-pick): show every state but done in the overview line. Rationale: the entry's motive is a pawl advanced by hand.
- Q5 (auto-pick): leave status strand removal at run end out. Rationale: not asked, and no path removes it today.
- Q6 (settled by the parent's and the arbiter's added entries): this bench arm designs against base f98a1ba0 and ignores parent commit `7c23591`, which lands a multi-overview design.
