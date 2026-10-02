---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (Decision 5, cap paragraph)
    summary: Decision 5 names two cap counters, the recipe attempts and the store count, which disagree, and never says which one fails the entry
  - id: F2
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (Decision 5, last Decision paragraph)
    summary: The Pending and Final relaxation has no stated key and no stated bound, and Final on an unconsumed reject passes today, so a session loss after the cap's reject lets the discussion through
  - id: F3
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md (Decision 5, way-out paragraph)
    summary: The way out ignores Discussion-Write's fresh-spawn BeginRound, so a resume opens a new round and asks the parent again instead of passing on the approve or halting without it
  - id: F4
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Decision 5, Bound)
    summary: The approve supersede bound does not say how a run halted at the cap is determined or that approve checks no caller identity either
  - id: F5
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Acceptance criteria, parent-review gate)
    summary: The acceptance criteria do not exercise the refusals Decision 5's bound promises
  - id: F6
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Decision 4)
    summary: Decision 4 does not say what a failed liveness read does, and a gate closure error ends the writer's shuttle run
  - id: F7
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Constraints, sixth bullet)
    summary: The no-Go-sends constraint is false as worded, since the shuttle engine already sends gate re-prompts and delivery prompts into the writer's session
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decision 4, Open risks first item)
    summary: That a nested pair's parent-review always ends in a timed pass is stated only in Open risks, not in the Decision that causes it
  - id: F9
    severity: LOW
    location: _lyx/discussion/decision-record.md (Decisions 4 and 7, Rationale)
    summary: The Rationale paragraphs of Decisions 4 and 7 re-argue rejected alternatives already covered by Scope Out, Constraints and the support log
  - id: F10
    severity: NIT
    location: _lyx/discussion/decision-record.md (Decision 5, Rationale)
    summary: Three does not match the discussion gate's budget, which re-prompts three times and fails on the fourth failure, while the cap fails on the third reject
---

### [BLOCKING] Decision 5 has two cap counters and does not say which decides

**Location:** `decision-record.md`, Decision 5: "The cap is 3 rejected rounds, set by the recipe entry's `attempts: 3`" and "The gate counts rejected rounds from the round store on disk".

**Issue:** In `shuttleengine`, `attempts` is a re-prompt budget held in memory (`run.gateFails`).
`handleGatedBoundary` (`internal/shuttleengine/wait.go`) finalizes a failing entry only when `gateFails >= Attempts`, and increments the count only after a re-prompt.
With `attempts: 3` the first three rejects each re-prompt, and only a fourth fails the entry, not the third.
The count also resets on attach (`GateEntry.PassOnCap`'s doc), so after an attach with two rejects in the store, the third reject is the engine's first failure and re-prompts.
A closure has no way to fail an entry while the engine's budget remains: a `Failed` result always re-prompts.
The record states both counters and never says which one fails the entry or how the store count overrides the engine's, so the plan writer cannot build "the third reject fails the entry".

**Fix:** State that the store count decides.
The entry's `attempts` is told to the gate as the cap.
A reject below the cap is an ordinary failure that re-prompts.
The reject that brings the store count to the cap returns a new terminal failure result that the engine finalizes as failed without a re-prompt, whatever its in-memory count.
The in-memory count never exceeds the store count, so the engine's budget cannot fail the entry first, and a must-pass entry is never let through.

### [BLOCKING] The Pending and Final relaxation has no key and no bound

**Location:** `decision-record.md`, Decision 5: "the shuttle engine accepts `Pending` from this entry, and its `Final` closure keeps reporting the latest round's state".

**Issue:** `evaluateGate` refuses `Pending` and a non-nil `Final` from any entry that is not `pass_on_cap`.
The engine knows entries only by their `GateEntry` fields, so "this entry" names no mechanism, whether entry name, a recipe key or a new field.
The Decision also says nothing about what the relaxation lets through.
`GateEntryWaiting` counts toward `Passed`, and parentreview's `final()` returns passed, with a Warn, on an unconsumed reject.
If the writer's session is lost or the deadline expires after the cap's reject lands but before the gate reads it, finalize runs `Final`, which passes, and the run goes on to Discussion-Review despite the third reject.
That bypasses the new guard, and the attack-surface rule requires the Decision to bound it.

**Fix:** Key the relaxation on a new `GateEntry` field that only the parent-review wiring sets, never an entry name or a recipe key.
State that it lets a must-pass entry hold the run and set `Final`, and lets nothing through: the entry passes only on an approve, the unchanged timeout expiry, no reviewer, or no open request.
`Final` fails on an unconsumed reject, which counts toward the cap as at `Gate`.

### [BLOCKING] The way out ignores the fresh-spawn round

**Location:** `decision-record.md`, Decision 5: "the next `lyx loom start` then passes the entry" and "Without that approval, a resume reaches the same failed gate and halts again".

**Issue:** The halted Discussion-Write session has ended, so a resume re-runs the row as a fresh spawn.
Its preparation (`internal/shedrecipe/entries_discussionwrite.go`) calls `Store.BeginRound` on every fresh spawn, which creates an empty round N+1.
On the next arrival the gate finds no request in the latest round and opens one.
After an approve, the parent is asked to review again instead of the entry passing.
Without an approve, the parent reviews a fourth round instead of the run halting again.
The same applies to F2's unconsumed reject: the re-spawned writer would never see the review.
Neither of the Decision's two claims holds unless it says how the fresh-spawn round and the cap check interact.

**Fix:** State that the fresh-spawn preparation keeps the latest round when it holds an unconsumed reject or a superseding approve, and otherwise opens the next round as today.
State that the gate reads the latest round before opening any request.
A superseding approve passes, the cap's reject fails at once without a new request, and an unconsumed reject below the cap is consumed and re-prompts with its review.

### [MEDIUM] The approve supersede bound is not mechanical

**Location:** `decision-record.md`, Decision 5, Bound.

**Issue:** "Approve's new supersede path reaches only the latest round of a run halted at the cap" does not say how the verb decides that a run is halted at the cap.
It could use the run's shed status, the store's reject count, the latest round's verdict, or all three.
The bound says reject checks no caller identity but is silent on approve, whose new path overrides a reject, so the reader cannot tell what limits who may supersede.

**Fix:** Define "halted at the cap" as all three together: the run's status is `blocked`, the store holds cap rejected rounds, and the latest round's verdict is the cap's reject.
State that approve, like reject, checks no caller identity, so the halted state is the only gate on the supersede path.
State that approve refuses otherwise and records the approve as superseding.

### [MEDIUM] The acceptance criteria skip the promised refusals

**Location:** `decision-record.md`, Acceptance criteria, parent-review gate item.

**Issue:** The bound promises that approve cannot supersede outside the cap or on an earlier round, and that a resume without approval halts again.
No criterion exercises any of these, nor the session-loss path F2 names.

**Fix:** Add criteria for each: approve on a run that is not halted refuses, as does approve naming an earlier round; a resume without approval fails the entry again without opening a request; a lost session after the cap's reject fails the entry; an attach between rejects does not move the cap.

### [MEDIUM] Decision 4 does not bound a failed liveness read

**Location:** `decision-record.md`, Decision 4.

**Issue:** The gate now reads reed state for the parent worktree on each evaluation.
The Decision does not say what happens when that read fails, for example because the parent worktree has no reed session or its state file is unreadable.
A closure error is returned verbatim from `evaluateGate`, and `handleGatedBoundary` ends the shuttle run with `shuttle: gate:`.
Left unstated, a liveness read error would end the writer's run instead of holding it.

**Fix:** State that a failed liveness read counts as not live: the gate holds, logs a Warn naming the error, and stays bounded by `parent_review_wait_min`.

### [MEDIUM] The sixth constraint is false as worded

**Location:** `decision-record.md`, Constraints, sixth bullet.

**Issue:** "no Go process sends into a Claude session except the orch watcher as today" contradicts existing behaviour.
The shuttle engine (Go) sends gate re-prompts and the parent-review delivery prompts into the Discussion-Write session through `Run.Send`, and Decisions 4 and 5 keep and extend those sends.
The rule the design means is about the orch session (Scope Out: "Any Go process typing into the orch pane").

**Fix:** Narrow it to the orch session: no Go process sends into the orch session except the orch watcher, and batten sends into no session.

### [LOW] The nested-pair timed pass lives only in Open risks

**Location:** `decision-record.md`, Decision 4; Open risks, first item.

**Issue:** Decision 3 resolves a nested pair's reviewer to an orch name in that pair, where `lyx orch` never runs.
Under Decision 4's hold, that pair's parent-review therefore always ends in the timed pass.
The wait bound does bound it, so this is not an unbounded guard.
But the Decision that turns the gate into a certain timed pass for a whole class of pairs does not say so, and the plan writer reads Open risks as risks, not behaviour.

**Fix:** State in Decision 4 that a reviewer which never becomes live, as for a nested pair, holds until the wait bound and then passes with the Warn.
Trim the Open risks item to the upward-message consequence.

### [LOW] The Decisions 4 and 7 rationales re-argue rejected alternatives

**Location:** `decision-record.md`, Decision 4 Rationale ("the operator deferred `creel`, so no store-and-forward mailbox") and Decision 7 Rationale (the pane-typing argument).

**Issue:** Both argue against options in the support log's Rejected alternatives: the queue and batten typing into the orch pane.
Completeness test: the requirement each carries, no mailbox and no Go typing into the orch pane, already sits in Scope Out and the sixth constraint.
Moving the argument loses nothing the plan writer needs.
Decision 2's rationale ("rather than adding a second source") is a single clause stating the chosen option's own reason and stays.

**Fix:** Cut the `creel` clause from Decision 4's rationale.
Cut Decision 7's rationale to batten having no SendMessage and the driver's message already carrying what the parent needs.
Move the pane-collision argument to the support log's rejected alternative.

### [NIT] "Three matches the discussion gate's own budget" is off by one

**Location:** `decision-record.md`, Decision 5, Rationale.

**Issue:** The `discussion` gate's `attempts: 3` re-prompts three times and fails on the fourth failure.
The cap fails on the third reject after two re-prompts.

**Fix:** Drop the comparison, or restate it as the number of rounds the parent reviews.
