---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D2, D4, D5
    summary: Tick-time gate re-evaluation and the delivery prompt leave unsettled what happens when a verdict or a re-send lands while the writer is mid-turn, so a reject can be consumed and silently lost
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D4, D5
    summary: The delivery re-send loop has no count bound on notices reaching the parent, and does not say whether a delivered --failed record stops it
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D6
    summary: D6 introduces verbs with no caller check but states no bound on what an unchecked approve, reject, delivered or notify lets through
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D5, D8
    summary: The writer is told to run a verb that writes outside its two files and to write into the gate-owned Review rounds section, but no Decision states the writer-stencil change that permits both
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/support-log.md Question ledger Q10, Interview turn 4; decision-record.md Acceptance criteria, Scope
    summary: Residue of the pre-hub-review reed design contradicts the revised D5 and D12
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md D5, Constraints (Agent Name Invariant)
    summary: D5 addresses the notice to $LYX_PARENT while the constraint says the seed's full name, without stating they are the same value
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md D5 Rationale, Auto-mode assumptions
    summary: Process history (hub-review attribution and date, revised-by note) sits in the decision record
---

### [BLOCKING] Mid-turn verdicts and re-sends are unsettled; a reject can be lost

**Location:** `_lyx/discussion/decision-record.md` D2, D4, D5.

**Issue:** D2 makes the wait loop re-evaluate the gate list on every poll tick while an entry is pending, and D5 makes the gate send the writer a delivery prompt, re-sent once a minute, which starts a writer turn that ends in a Done arrival.
The existing gated branch in `internal/shuttleengine/wait.go` evaluates the gate only on a Done arrival, and its comment rests on that ("the gate fires at a turn boundary, when there is no in-progress turn").
`Run.Send` (`run.go:521`) checks only that the TUI is up (`requireReadyAgentPane` → `StartupReady`), not that the writer is idle, and a failed re-prompt send ends the loop and finalizes.
The record does not say whether tick-time evaluation, the delivery prompt, or a reject re-prompt waits for a turn boundary.
Concrete failure: the reviewer submits reject while the writer is in a delivery-prompt turn.
The tick-time closure marks the reject consumed (D4) and the loop sends the re-prompt mid-turn.
Either the send fails, and the loop finalizes with the reject already consumed, so the review is dropped;
or the text queues behind the current turn, and that turn's Done arrival re-evaluates, finds the reject consumed, passes, and finalizes and commits the discussion while the writer is still working the review.
The record also does not say how a delivery turn's Done arrival interacts with the gate list: whether it re-runs `discussion` and what that does to the counts.
Separately, D4 has the closure "attempt delivery" and D5 has "the gate" send, but a gate closure has no handle on the live session; which component sends decides who knows the turn boundary.

**Fix:** State in D2/D5 that tick-time evaluation and every send (delivery prompt, reject re-prompt) happen only while the writer is at a turn boundary, meaning its last arrival was Done and no turn has started since;
that the closure returns the delivery prompt with its pending result and the wait loop sends it;
that a delivery turn's Done arrival is an ordinary arrival (it re-runs `discussion`, whose pass resets nothing that matters, and reaches the still-pending entry);
and that a failed delivery-prompt send is a Warn that leaves the entry pending rather than ending the loop.
Add an acceptance criterion for a reject submitted while the writer is mid-turn.

### [MEDIUM] The re-send loop has no count bound and an unclear stop condition

**Location:** `_lyx/discussion/decision-record.md` D4 (third bullet), D5.

**Issue:** D4 re-sends "while no delivery is recorded", and D3 makes the delivery record hold either a delivered-at time or the last failure reason.
Whether a `delivered --failed` record stops the re-send is unsaid; the acceptance criterion says only that it "leaves the run waiting".
A writer that sends the SendMessage but never stamps delivery gets a fresh delivery prompt every minute, and each one can produce another notice to the parent: up to 60 notices at the default bound, and each one costs the hub a fork if it obeys D12.
D5's bound covers where delivery lands, not how many notices it produces, so the new routing edge is unbounded in count.

**Fix:** Cap the delivery prompts per round (for example three: the first plus two re-sends), after which the gate Warns and stops re-sending, with `notify` still able to send one more.
State that a `--failed` record keeps the re-send going within that cap, since a failure reason is not a delivery.
Tell the reviewer brief or D12 that duplicate notices for one request are handled once.

### [MEDIUM] D6's unchecked verbs state no bound

**Location:** `_lyx/discussion/decision-record.md` D6, last decision line.

**Issue:** "The verbs do not check who runs them" removes any identity guard, and the writer already runs `lyx loom review delivered` from its own worktree, where `<slug>` is optional.
Nothing stops the writer from running `approve` on its own discussion, or stamping `delivered` without sending.
The rubric's attack-surface question requires the Decision to say what this lets through and what bounds it; D6 does not.

**Fix:** State the bound in D6: an unchecked `approve` or `delivered` can skip at most this one advisory review, which is pass-on-cap and which wait-bound expiry skips anyway;
the independent Discussion-Review rounds still run;
`reject` can at most spend the single pass-on-cap re-prompt;
`notify` only re-sends within the D5 cap;
and the committed request directory keeps every verdict visible to Discussion-Review and the operator.

### [MEDIUM] The writer stencil's write boundary is not opened

**Location:** `_lyx/discussion/decision-record.md` D5, D8.

**Issue:** The writer stencil (`loom-template-discussion.md`) says the only files the writer creates or modifies are the two discussion files, that everything else is read-only, and that `## Review rounds` is seeded `_No rounds yet._` for the Discussion-review gate to append to.
D5 has the writer run `lyx loom review delivered`, which writes the request directory, and D8 has it write a parent-review entry under `## Review rounds`.
D8 says only that the stencil "gains a short paragraph describing this re-prompt"; no Decision states that the write boundary and the Review-rounds ownership change, so the plan writer may leave a stencil that forbids what the delivery prompt asks.

**Fix:** State in D8 that the writer stencil's "What you may write" section gains two carve-outs: running `lyx loom review delivered` when a delivery prompt asks, and appending one parent-review entry to `## Review rounds` when a parent-review re-prompt asks.
Add it to the stencil acceptance criterion.

### [MEDIUM] Residue of the reed design contradicts the revised D5

**Location:** `_lyx/discussion/support-log.md` Question ledger Q10 and Interview turn 4;
`_lyx/discussion/decision-record.md` Acceptance criteria (last bullet) and Scope.

**Issue:** Q10 says a hand-started hub "gets no push notice under this design" and is Open, but the revised D5 reaches a hand-started hub through SendMessage.
Interview turn 4 still answers with reed `SendText` and hub-wide name resolution, cited as D5, with no note that it was replaced (turn 5 has one).
The acceptance criteria ask for unit tests of "the hub-wide name resolution", which only the reed design needed; nothing in the revised record resolves names hub-wide.
Scope names "the orch start stencil" while D12 changes the start and resume stencils.

**Fix:** Resolve Q10 under the SendMessage design;
mark turn 4's answer as replaced by the hub review;
drop the hub-wide name resolution test;
make Scope name both orch stencils.

### [LOW] Two names for the delivery address

**Location:** `_lyx/discussion/decision-record.md` D5 and the Agent Name Invariant constraint.

**Issue:** D5 renders the reviewer's full name into the delivery prompt and also tells the writer to send to `$LYX_PARENT`; the constraint says the reviewer is addressed by the full name the seed records.
`internal/hubgeom/hubgeom.go` sets the task worktree's `ParentName` from its default run's seed, and reed exports that as `LYX_PARENT`, so they are the same value, but the record does not say which one the prompt addresses or that they agree.
D4 already passes the review when the seed has no parent, so the empty case never reaches delivery.

**Fix:** Have the prompt address the rendered full name from the seed, and note that `$LYX_PARENT` is reed's export of the same seed field.

### [NIT] Process history in the decision record

**Location:** `_lyx/discussion/decision-record.md` D5's "Rationale (hub review, 2026-10-02)" and the first Auto-mode assumption's "(revised by the hub review)".

**Issue:** The attribution and date are deliberation history; the support log's Review rounds already carries them.
The rationale's content (the hand-started hub is not a strand; SendMessage already carries driver notices; the operator chose Claude Code's messaging over a lyx mailbox) is what the plan writer needs and stays.

**Fix:** Keep the rationale's content and drop the attribution, date and "revised by" note.
