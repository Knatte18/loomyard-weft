---
round: 1
focus:
  - "internal/shuttleengine/activity.go, ReadAgentActivity and noteWaiterlessRun: does a run that reads running with a dead waiter log one Warn, then Debug lines for the rest of the episode, and does the episode end with one Info once the run reads live or no longer running? Check `logger.EpisodeSet` Fail/Recover semantics against the acceptance criteria."
  - "The not-found and no-pid paths: does a record with no pid, and a record with a terminal outcome, log nothing new, including no stray recovery Info when no episode was open?"
  - "A run that goes from waiterless back to live: is the recovery line logged exactly once on the next read, and do the unreadable-record and unknown-engine episodes still behave as before?"
  - "Scope against the decision record: the diff must not change which runs count as live, the notice text, or Batten's quiet judgement, and must not change the run record's shape."
  - "Docs in the same diff: the `ReadAgentActivity` doc comment and `internal/shuttleengine/doc.go` match the new behavior, and PATTERN-log-levels is followed."
  - "internal/shuttleengine/activity_test.go: does the new test cover Warn-then-Debug, the recovery Info, and the no-pid and finished-record cases, without duplicating an existing test's coverage?"
---

Seed focus from the decision record's acceptance criteria and constraints.
