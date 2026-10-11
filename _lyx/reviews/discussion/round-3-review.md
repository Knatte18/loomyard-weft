---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:65
    summary: D2's Bound says a resume of the Stuck-halted run expires the open request at its first boundary, but finalize removes the strand on any OutcomeDone, so no attach is possible and the next start is a fresh spawn that begins a new round
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: D2's Rationale says finalize stays the one place an expired request is marked, but the gate closure's own expiry branch marks it at a boundary, which D2's Bound itself relies on
---

### [MEDIUM] D2's Bound claims a resume path that does not exist after a Stuck halt

**Location:** `_lyx/discussion/decision-record.md:65` (D2, Bound, the "A resume of the halted run" sentence)

**Issue:**
D2's Bound says that when finalize stops at a failing mechanical entry, "A resume of the halted run marks that request expired, with the timeout Warn, at its first boundary whose tree passes the mechanical entry, through the gate's existing expiry branch."
The code gives no such resume.
The probe finalizes `OutcomeDone`, and `finalize` in `internal/shuttleengine/wait.go` sets `cleaned := outcome == OutcomeDone && !run.spec.KeepPane`, removing the strand and the run directory whatever the gate outcome.
`DiscussionSpec` in `internal/loomengine/discussion.go` leaves `KeepPane` false, and the seats chair gets `KeepPane: !isChair`, also false.
`shedadapters.SingleLLMProducer` maps the failing gate to `Stuck` (`internal/shedadapters/singlellm.go`, the `result.Gate != nil && !result.Gate.Passed` branch), and Discussion-Write has no `on_stuck`.
On the next `lyx loom start`, `AttachGated` finds no live, never-terminated run, so the producer takes the fresh-spawn path.
That path runs `PrepareRound` (`internal/shedrecipe/entries_discussionwrite.go`).
The latest round holds an open request with no verdict, which neither the existing reject and superseding-approve cases nor D9's "holds no request" case reuses, so `BeginRound` starts round N+1.
The gate reads only the latest round, so round N's request is never expired and no timeout Warn is logged for it.
The next sentence, "A fresh spawn leaves it open in its round and works on the round `PrepareRound` begins", describes what happens.
The "resume" sentence describes a branch that has no route to it.
`evaluateGate` itself matches D2's other claims: entries run in list order, a failing entry stops the walk, so `parent-review`'s `Final` is not reached and the request stays open and unexpired with no Warn.
The acceptance line at `decision-record.md:207` matches that code.
The sentence came in with the fix for round 2's F1, which read "resume" as a live attach that `gate()`'s expiry branch could reach.
The false sentence is the plan writer's only statement of what recovery does after the halt, so a plan could pin a test or add code for an expiry-on-resume that the design never builds.
D2 states no change to `evaluateGate` or to finalize's cleanup, so the third focus item, which asks for the attack surface of an edit to the halt behavior, has no edit to judge.

**Fix:**
Delete the "A resume of the halted run…" sentence.
State that the halt removes the writer's strand, so any later start is a fresh spawn: round N's request stays open and unexpired, and the new spawn works on round N+1.
If the orphaned open request matters, for example to a reviewer who later records a verdict meant for round N against the latest round, add that consequence to D2's Bound or to Open risks.

### [LOW] D2's Rationale says finalize is the only place that marks expiry

**Location:** `_lyx/discussion/decision-record.md:71` (D2, Rationale, second sentence)

**Issue:**
"finalize stays the one place an expired request is marked" contradicts the code and D2's own Bound.
`closures.gate` in `internal/parentreview/gate.go` calls `s.MarkExpired()` and logs the timeout Warn when `now.Sub(r.Request.OpenedAt) >= c.cfg.WaitBound` at a boundary evaluation, separately from `final`.
D2's Bound names that path as "the gate's existing expiry branch".

**Fix:**
Narrow the claim to what D2 keeps: the probe marks nothing, and expiry is marked only by the parent-review closures, at a boundary or in `Final`.

Focus item 2, D11: `parentreview.Delivery` holds `delivered_at`, `failed_reason`, `prompts`, `last_prompt_at`, `waiting_notifies` and `cap_warned`, and `Verdict` holds `kind`, `recorded_at` and `consumed`.
`internal/loomcli/review.go` says the verbs check no caller identity.
This run's `_lyx/reviews/parent-review/round-1/delivery.json` has `delivered_at` 19:52:27, and `verdict.json` has an approve with `recorded_at` 19:52:43.
D11's Decision, its landing caveat ("neither records who stamped the delivery, that a SendMessage to `ly:orch` preceded the stamp, who ran the verdict verb or which notice prompted it"), its Rationale and its acceptance line all match what those two files record.
No finding there.
