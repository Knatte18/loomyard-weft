---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:D6
    summary: D6 says a re-prompt that did not land is tried again on a later boundary the way a failed pending send is, but nothing retries it, so the run idles to its deadline
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D7 and Constraints (PATTERN-completion-signal)
    summary: The stopped outcome is outside attach's file-contract-first check, so the Constraint to keep that precedence is vacuous; the Decision never says whether stop consults allOutputFilesExist
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D1 and Open risks
    summary: D1's turn_start-after-turn-end condition holds a send against an idle pane after an interrupted turn (no turn end), and Open risks wrongly claim a missing hook line never causes a false busy
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D4
    summary: D4 states the premise that an agent whose send did not land has done no work, but a late-consumed prompt can start work before abandonStartup removes the strand, and no bound is stated
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D2 and D3
    summary: D2 says the settle wait counts against D3's window, but that window starts after Enter; as written the settle loop has no bound
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:D7 and Acceptance criteria
    summary: D7 lets a concurrent Wait overwrite stopped while an acceptance criterion requires the burler half to be recorded stopped; which wins is undecided
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D7
    summary: D7 does not say what the stop verb does when writing the stopped record fails, which is the order the Rationale says must not be reversed
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:Acceptance criteria
    summary: Acceptance criteria omit behaviours the Decisions state, and D4 leaves the finalize outcome unnamed
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:Question ledger Q8
    summary: Q8 leaves the widened TransientAgentStart definition open for the operator, but D4 treats it as decided and Open risks does not mention it
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:Auto-mode assumptions and Notes for the plan writer
    summary: The defaults bullet restates D1, D3, D5 and D9, and the incident chain restates the D4 and D7 rationale and support-log interview 1 and 2
---

### [BLOCKING] D6's retry of a failed re-prompt never fires

**Location:** `decision-record.md` D6 ("leaves the entry at the boundary ... tried again on a later boundary, the way a failed pending send already is"), Notes for the plan writer.
**Issue:**
The analogy is false.
In `wait.go` a failed pending send is retried every tick because the loop has an explicit `else if run.gatePending && run.gateAtBoundary` branch, and `handleGatedBoundary` returns early on `gatePending`.
A failed re-prompt is not pending: `gateFailedAt` is set and `gatePending` is false.
The only way into `handleGatedBoundary` for it is a new `OutcomeDone` from `pollEventsTick`, which needs new events bytes, and `pollEventsTick` returns no outcome when nothing new was read.
The writer got no re-prompt and is idle at its boundary, so no new turn end arrives.
The entry stays at the boundary until the run deadline, which then reaches `classifyDeadlineExpiry`.
The goal "when it cannot, the run retries instead of halting" is not met for the gate's re-prompt, and the Bound ("bounded by the run deadline") describes the failure rather than a retry.
Each retry that does run also blocks the loop inside `Send` for up to `send_ready_timeout_s` plus `submit_confirm_timeout_s`, so the deadline is checked only between sends.
The Decision states no cap on that overshoot.
**Fix:**
Decide the retry trigger: a failed re-prompt marks the boundary pending-like so the existing per-tick branch re-attempts it, or the Decision names another trigger.
State that the deadline is checked between sends and that a send's own windows can overshoot it by their sum, or state a cap.
Add an acceptance criterion that pins the retry without a new writer event.

### [MEDIUM] `stopped` sits outside the file-contract-first check

**Location:** `decision-record.md` D7 and Constraints (`PATTERN-completion-signal`).
**Issue:**
In `attach.go`'s `dispositionCandidate` the file-contract-first check fires only for `Outcome == runOutcomeRunning`.
For a record whose strand is tracked and not live without a cleared binding, the function returns `verdictRespawnEligible` for any non-running outcome without consulting `allOutputFilesExist`.
A `stopped` record therefore never meets the check, and the Constraint "attach's file-contract-first checks keep their precedence over the new `stopped` outcome" has nothing to keep.
D7 also writes `stopped` over a record still reading `running`.
If that run's outputs already exist, the write removes the one state in which attach would harvest it as finished.
`PATTERN-completion-signal` lists the negative answers and pins the audited sites in `completionsignal_enforcement_test.go`, so a new site that writes a terminal record is a decision the Constraints must name.
**Fix:**
Decide whether `stopped` means "lyx discarded this run's work", in which case say that attach does not harvest its outputs and the Constraint is reworded to say so.
Otherwise state that the stop verb consults `allOutputFilesExist` first and leaves the record alone, or writes `done`, when the outputs exist.
Say which side of the `PATTERN-completion-signal` tripwire the new sites fall on, so the plan writer updates the pinned list in the same commit.

### [MEDIUM] D1's turn-start condition can hold a send against an idle pane

**Location:** `decision-record.md` D1 (signal condition), Open risks last bullet.
**Issue:**
The condition holds the send while a `turn_start` follows the newest turn end.
An interrupted turn leaves a `turn_start` with no turn end: `readSessionFacts` in `sessionread.go` and the `SessionProber.TurnStartInterrupt` capability exist because the repo already treats that case as real.
A crashed session with a missing `Stop` line is the same shape.
The pane reads idle, the signal condition never clears, and the send fails with `ErrSessionBusy` after the full window.
At start that becomes a not-started teardown (D4), mid-run it becomes the D6 loop.
Open risks claims only that a missing hook line "falls back to the pane reading alone, never to a false busy".
That holds for a missing `UserPromptSubmit` line and not for a missing turn end.
**Fix:**
State the bound for an unmatched `turn_start`: either the pane reading overrides the signal once the pane has read idle for a stated period, or the interrupted-turn reading from `TurnStartInterrupt` clears the hold.
Correct the Open risks bullet to name the missing-turn-end case.

### [MEDIUM] D4's "no work done" premise has no bound

**Location:** `decision-record.md` D4 (Rationale and Bound), focus directive for D3 and D4.
**Issue:**
After `ErrSubmissionNotLanded` the text may still be sitting in the box, or a late Enter may still be in flight.
`abandonStartup` finalizes the record and then removes the strand, so the pane can consume the prompt and start a turn in between.
The Rationale ("has done no work, so re-running the step unchanged can succeed") rests on that not happening, and the Bound covers only the class set.
If the agent wrote a partial output file before removal, the re-step's `Spec.validate` refuses a spec whose output file already exists, so the retry fails instead of proceeding.
**Fix:**
State the bound in D4: for example, that teardown re-reads the box and a `turn_start` signal before removal and treats a landed prompt as a started run, or that removal happens before any further send and the window is accepted as a known residual with the retry's validate-refusal named.

### [MEDIUM] D2's settle wait and D3's window disagree

**Location:** `decision-record.md` D2 ("The settle wait counts against the submit window of D3") against D3 ("after Enter ... until `submit_confirm_timeout_s` has passed").
**Issue:**
D3's window starts at Enter and D2's settle runs before Enter, so it cannot count against it.
D2 reads the box until two consecutive reads agree and states no other bound.
A box whose content keeps changing (an animated placeholder, a streaming paste) never settles, and the loop has no exit.
**Fix:**
Give the settle read its own bound, or define one window that starts at the first typed character and covers D2 and D3, and say which error a settle that never converges returns.

### [MEDIUM] Overwrite of `stopped` against the burler acceptance criterion

**Location:** `decision-record.md` D7 ("a concurrent `Wait` ... may overwrite `stopped` ... either answer is terminal") against the Acceptance criterion "its first half stopped and recorded `stopped`".
**Issue:**
`finalize` writes the outcome from the in-memory run state unconditionally.
The stopped half's own `Wait` then writes its outcome over `stopped` whenever it finalizes: through the file contract (`finishedDespiteMechanismFailure`), on a timeout, or on a dead pane.
The record can read `stopped` only if that `Wait` ends in a mechanism-failure error and never finalizes, which the Decision does not say.
The Rationale rejects `died` because it hides that lyx stopped the run on purpose, and the overwrite brings that back.
The stop verb called through a run handle should also update the handle's own state, or its next save restores `running`.
**Fix:**
Decide: either the stop verb marks the in-memory run so a later `Wait` finalize keeps `stopped` for the non-`done` outcomes, or the acceptance criterion weakens to "terminal, and `stopped` unless the half's `Wait` finalized first".
State which, and who owns the handle update.

### [LOW] Stop verb: failed record write

**Location:** `decision-record.md` D7.
**Issue:**
The Rationale says the record must be written before the removal.
The Decision and criteria cover a removal failure and say nothing about a record-write failure: abort without removing, or remove anyway, which would recreate the #444 state.
**Fix:**
State the order of failure handling: a record write that fails returns an error and removes nothing, or removes and returns the error with the record left `running`.

### [LOW] Acceptance criteria gaps and an unnamed outcome

**Location:** `decision-record.md` D4, D5, D7, D9, Acceptance criteria.
**Issue:**
No criterion pins: the guid-only stop with no record (D7); an already-terminal record keeping its outcome (D7); `Runner.Send` waiting for idle (Q6); a start failure past `AddStrand` such as the prompt-offset persist taking `abandonStartup` (D4); the 15-line cap and the failed-capture message (D5); the zero-or-negative handling being per key (D9).
D4 says the record is "finalized to a terminal outcome" without naming it, while the burler criterion says `died`.
**Fix:**
Add the criteria as rows of the existing behaviours, and name the D4 outcome.

### [LOW] Q8 stays open while D4 treats the question as decided

**Location:** `support-log.md` Question ledger Q8; `decision-record.md` D4, Open risks.
**Issue:**
The ledger marks the widened `TransientAgentStart` definition open for the operator, and the decision record states it as built and mentions no pending decision.
`Plan-Write` never reads the ledger.
**Fix:**
Either resolve Q8 as an auto-pick, or add an Open risks line that D4's class widening awaits the operator.

### [NIT] Restated content in the decision record

**Location:** `decision-record.md` Auto-mode assumptions (defaults bullet), Notes for the plan writer (incident chain).
**Issue:**
The defaults bullet repeats numbers D1, D3, D5 and D9 already state; the incident chain repeats the D4 and D7 rationale and the support log's Interview 1 and 2.
The completeness test applies: the code pointers in the Notes (`loadSkillsThenPrompt`, `handleGatedBoundary`, `signals.go`) carry what the plan writer needs, but the chain narrative does not.
**Fix:**
Drop the defaults bullet and the incident-chain item; keep the code pointers.
