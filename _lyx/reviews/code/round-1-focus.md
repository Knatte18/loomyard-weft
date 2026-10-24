---
round: 1
focus:
  - "internal/shuttleengine/doc.go after the parent merge: does the new waiterless-run paragraph in the Agent activity section sit coherently beside the parent's usage-limit sentences in the held-turn-end section, with no merge damage or contradiction?"
  - "internal/shuttleengine/activity.go `ReadAgentActivity` and `noteWaiterlessRun`: does every path that ends a waiterless-run episode (live again, terminal outcome, no pid, vanished record) call Recover on the run's own key, and does a repeating skip log Warn once and Debug after, per PATTERN-log-levels?"
  - "internal/shuttleengine/activity_test.go: does each acceptance criterion in the decision record (Warn once, Debug on repeat, Info on live again, nothing for no-pid or finished records) have a test, without duplicating another test's coverage?"
  - "Scope against the board entry and record: does the diff leave which runs count as live, the notice text and Batten's judgement unchanged, and keep the run record's shape as is?"
---

Seed focus for the amendment round: the parent merge touched only `internal/shuttleengine/doc.go`, so the doc paragraph is the first place to look.
The remaining entries settle the logic and test coverage against the decision record's acceptance criteria.
