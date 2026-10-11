---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md` D2 (Bound and Rationale), settle whether every sentence about what happens after the probe's gate halts at a failing mechanical entry matches the code: `finalize` in `internal/shuttleengine/wait.go` removes the strand and run directory on any `OutcomeDone` when `KeepPane` is false, `shedadapters.SingleLLMProducer` maps a failing gate to `Stuck`, and `PrepareRound` in `internal/shedrecipe/entries_discussionwrite.go` decides whether a later start begins a new round. Check that no sentence describes a resume, attach or expiry route the code has no path to."
  - "In D2, settle whether the stated consequence of an open, unexpired request left behind by the halt is bounded: what a reviewer recording a verdict afterwards can reach, given the gate reads only the latest round, and whether the Decision, Bound or Open risks names that bound."
  - "In D2's Rationale, settle whether the claim about where an expired request is marked agrees with `closures.gate` and its final step in `internal/parentreview/gate.go`, which mark expiry by the wait-bound comparison at a boundary evaluation and in `Final`."
---

Rubric reminders: an absent optional plan-writer subsection, absent rejected alternatives in `decision-record.md` and incomplete call-site enumeration are never findings.
