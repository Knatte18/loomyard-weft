---
round: 1
focus:
  - "Decisions 3 and 4 together: Decision 3 resets a passing entry's count to zero, and Decision 4 claims a PassOnCap entry fires at most Attempts re-prompts per run. Settle whether an entry that fails, passes, then fails again breaks that bound, and whether the 'one-time request' framing in the Goal holds."
  - "Acceptance criteria's 'existing tests pass with only the spec-construction syntax changed' against Decision 5: shuttleengine gate tests build GateSpec{Gate: gate} with no Attempts and rely on today's Attempts<=0-means-3 default. Settle whether that criterion is achievable as written once 0 means off, or whether the record must say those tests gain an explicit budget."
  - "Decision 6's definition of Passed false (exhausted, re-prompt send failed, session lost, deadline) against today's GateOutcome behaviour: settle whether 'failing at the final arrival' is well defined when the run ends between arrivals (after a re-prompt, before the agent settles again), and whether single-gate rows keep identical Passed/Attempts/FindingsPath in those endings as the acceptance criteria require."
  - "Decision 2 names finalize's verdict site as an arrival: settle whether per-gate counting, the PassOnCap let-through, and the five per-gate states are specified for that site, or only for Wait's gated-Done branch."
  - "Refusal Way-Forward handling: Constraints binds new reachable construction errors to refusal-spec rows, while the Auto-mode assumptions and ledger item 12 defer reachability to Plan, and the acceptance criteria name no refusal-spec work. Settle whether the record gives the plan writer enough to act on either outcome."
  - "Out-of-scope's attach paragraph prescribes how parent-review's review gate should make its closure idempotent. Apply the completeness-before-leanness test: settle which part is a constraint this task's plan writer needs and which part is design for another task that belongs outside this record."
  - "Code-level detail in decision-record.md (Decision 1's hop list and nil-test rewrite, Decision 9 and the Notes' function names) against the writer stencil's bounded-exploration rule: settle whether any of it is signature- or call-site-level material that does not belong in Discussion, without flagging the optional Notes section's existence or its incompleteness."
---

Seeded from a first read of both artifacts, the writer stencil, and a spot check of `internal/shuttleengine/gate.go`, its tests, `CONSTRAINTS.md`, and `contracts/specs/refusal-spec.md`.
The entries are where to look, not verdicts; the round decides each one.
