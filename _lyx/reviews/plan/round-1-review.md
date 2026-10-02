---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/02-parent-resolver.md:27
    summary: The resolver's prime check stats the parent worktree's .git entry and returns the error, so a pair whose parent pair has been removed fails card 8's loom wiring outright instead of resolving a name
  - id: F2
    severity: LOW
    location: _lyx/plan/05-review-gate-rounds.md:31
    summary: The final closure's below-cap reason says the re-spawned writer is re-prompted with the review, which is false for a consumed reject, where card 5's own arrival table opens the next round instead
  - id: F3
    severity: LOW
    location: _lyx/plan/07-parent-review-recipe.md:23
    summary: Decision 5's claim that the in-memory count never exceeds the store count holds by construction across cards 3, 5 and 7, but no card states it, names the precondition that GateEntry.Attempts equals GateConfig.Cap, or tests it
  - id: F4
    severity: LOW
    location: _lyx/plan/06-review-gate-hold.md:18
    summary: A failing ReviewerLive read logs a Warn on every evaluation, and a pending entry is evaluated on every poll tick, so a persistent liveness error floods the log where the hold's Info line is bounded to once per round
  - id: F5
    severity: LOW
    location: _lyx/plan/10-review-approve-supersede.md:28
    summary: A second reject on a run halted at the cap still gets the verdict-recorded way forward saying the run proceeds on its own, which the cap makes false; card 10 leaves that message and its refusal-spec row unchanged
---

### [MEDIUM] Removed parent worktree turns the resolver into a hard wiring error

**Location:** `_lyx/plan/02-parent-resolver.md:27`, `:34`; `_lyx/plan/08-loom-reviewer-resolved.md:22`

**Issue:** Card 2 decides the prime form by calling `isPrimeWorktree(filepath.Join(l.HubPath, ParentWorktree))` and says prime-check read errors are returned.
`isPrimeWorktree` (`internal/hubgeom/hubgeom.go:47`) stats `<root>/.git` and returns an error when it is missing.
A nested pair whose parent pair was removed with `lyx fabric remove` has `parent_worktree` set to a worktree that no longer exists, so `ResolveParent` errors.
Card 8 makes a resolver error a wiring error, and `newParentReviewConfig` runs inside `wire` (`internal/loomcli/wiring.go:524`) on every loom invocation, so the child's whole run can no longer be wired.
Decision 3 says the name is derivable from the worktree name and the hub shortname, both stable, and is exported with no liveness check; a parent worktree's existence is a liveness question the resolver should not depend on.

**Fix:** In card 2, state that a parent worktree with no `.git` entry is not the prime (the prime is the repository's main worktree and exists while any pair does), so it resolves to the pair form `<shortname>:<ParentWorktree>:orch`; only other prime-check errors are returned.
Add a resolver test row for an absent parent worktree.

### [LOW] Final closure's below-cap reason misdescribes a consumed reject

**Location:** `_lyx/plan/05-review-gate-rounds.md:31`

**Issue:** Failing the final closure on a consumed reject fits Decision 5: its "passes only on" list (approve, expired request, no reviewer, no open request) excludes a consumed reject, and the writer's rewrite after it was never reviewed.
The reason text is the defect.
It says the re-spawned writer is re-prompted with the review on `lyx loom start`.
For an unconsumed reject that is right: card 7's `PrepareRound` keeps the round and card 5's arrival table consumes it and re-prompts.
For a consumed reject the same table runs `BeginRound` and opens the next round's request, so the writer is never re-prompted with that review; the parent reviews the re-spawned writer's discussion as a new round.
That line becomes the run's `blocked` reason, so the operator reads the wrong account of what resume does.

**Fix:** Split the below-cap line: an unconsumed reject says the re-spawned writer is re-prompted with the review; a consumed one says the re-spawned writer's discussion goes to the parent as the next round.
Have the final-closure test assert each reason.

### [LOW] The in-memory-versus-store budget claim is carried by no card

**Location:** `_lyx/plan/07-parent-review-recipe.md:23`; `_lyx/plan/03-gate-hold-terminal.md:27`; `_lyx/plan/05-review-gate-rounds.md:22`

**Issue:** Decision 5 says the in-memory count never exceeds the store count, so the engine's own budget never fails the entry first.
The cards guarantee it: card 3 does not increment for a terminal failure, a gate error finalizes with no increment, card 5's only non-terminal failure is consuming an unconsumed below-cap reject, each such consume is a distinct rejected round, and `handleGatedBoundary` (`internal/shuttleengine/wait.go:344`) re-prompts while the count is below `Attempts`.
After the second below-cap reject the in-memory count is at most 2, below an `Attempts` of 3, and the third reject is terminal.
The guarantee rests on `GateEntry.Attempts` and `GateConfig.Cap` being the same value, and on no other non-terminal failure being added to the closure later.
Card 7 says only that `attempts` is told as `Cap`; no card states the claim or its precondition, and no test pins it.

**Fix:** In card 7's Intent, state that the built entry's `Attempts` and the gate's `Cap` are the same `attempts` value, and that because each in-memory increment consumes a distinct rejected round the engine's budget is never spent before the store reaches the cap.
Add a card 5 or card 7 test where two below-cap rejects each re-prompt and the third fails terminal rather than budget-spent.

### [LOW] Liveness-error Warn has no rate bound

**Location:** `_lyx/plan/06-review-gate-hold.md:18`

**Issue:** Card 6 logs the hold's Info line once per round but a `ReviewerLive` error as a Warn with no bound.
While the entry is pending, `handleGatedBoundary` runs on every poll tick with the writer idle (`internal/shuttleengine/wait.go:314`), so a persistent error, such as an unresolvable parent worktree in card 8's closure, logs one Warn per poll until the wait bound.

**Fix:** Log the liveness-error Warn once per round, or once per distinct error text within a round, remembered beside the Info line's last-logged round; add the case to the hold tests.

### [LOW] Verdict-recorded way forward is wrong at the cap

**Location:** `_lyx/plan/10-review-approve-supersede.md:28`, `:38`

**Issue:** `reviewWaySettled` (`internal/loomcli/review.go:35`) and the refusal-spec row `review: verdict already recorded` (`contracts/specs/refusal-spec.md:148`) tell a parent re-running `reject` that the round is settled and the run proceeds on its own.
With the cap, a run whose latest round is the cap's reject is halted `blocked` and proceeds only after `lyx loom review approve <slug>`.
Card 10 rewrites the help text but leaves this way forward and its row as they are.

**Fix:** In card 10, extend the verdict-recorded way forward and its refusal-spec row to say that a run halted at the reject cap proceeds only after `lyx loom review approve <slug>`.

## Focus verdicts

1. Card 5's final closure failing on a consumed reject fits Decision 5's "passes only on" list; not a defect.
   The below-cap reason is inaccurate for a consumed reject (F2).
2. The cards together guarantee the in-memory count never exceeds the store count on every incrementing path; no card's Intent carries the claim (F3).
3. Card 10's status reader is built over the worktree the existing pre-run already resolves from the slug (`resolveReviewTarget`), which suffices.
   Its bound (status `blocked`, store at the cap, latest round the cap's reject) is the full set Decision 5 names, and the `--review` refusal only narrows it.
   It states what `reject` can now halt.
   Not a defect.
4. Every doc in the Scope list is carried: `fabricengine` (card 1), `parentreview` (cards 4, 6), `shedrun` and the Agent Name Invariant and #341 (card 9), `battenshed`, the batten recipe comment, `docs/overview.md` and #338 (card 11), the loom recipe comment (card 7), and the `lyx loom review` help (card 10).
   `manifest/designs/` holds no module doc for these packages, and the module table and execution stack in `docs/overview.md` do not change, so no further doc is owed.
5. Cards 5 and 6 are separately reviewable and testable: card 6 adds a told seam and a hold inside the open-request branch, testable with the seam alone, and implements Decision 4 rather than Decision 5.
   Not a defect.
6. Card 3 bounds `MayHold` (lets nothing through, pending at a final evaluation is not passed), `Terminal` (only on a failing result, else a gate error; skips the re-prompt and the in-memory budget, and can only end in failure) and `Reason`.
   The terminal path finalizes `OutcomeDone` through the existing call, so the Completion Signal Invariant claim holds.
   `CONSTRAINTS.md` has no text on only `PassOnCap` entries pending or on gate finalization, so no constraint change is owed.
