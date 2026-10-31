---
round: 1
focus:
  - "internal/shuttleengine/carryover.go `runTurnStarts` and `operatorInstructions`: does the refactor onto `positionedTurnStarts` leave the carry-over reading the same instructions as before (acceptance criterion D1), including the prompt-turn-start detection, turn starts whose raw line is not found, and the clamped prompt offset?"
  - "internal/shuttleengine/hold.go `operatorStartedTurn`: is the newest-turn-start-before-the-turn-end choice (D2) correct against `held.offset`, which is the offset past the turn end's line, and `runTurnStart.end`, the offset past the turn start's line? Check the `>=` boundary, a prompt-only run, and a run with no turn start before the turn end."
  - "internal/shuttleengine/wait.go held branch: does the new case sit correctly among the usage-limit, background-wake, API-continue and interactive/quiet-hold/no-notifier cases (D3)? It must send and defer nothing, leave `RunState.NotifiedOffset` alone, keep the hold, wait mark and deadline, and let a later lym-started held end still be notified."
  - "Test economy in hold_test.go, carryover_test.go and fakes_test.go: does each acceptance criterion have a row in the existing tier0 hold-notice table test, and does any new row or fake change duplicate existing coverage? Is any added behavior left uncovered?"
  - "Doc and comment compliance: does the doc.go hold-notice paragraph state the new rule accurately and drop the old 'cannot tell' claim? Do the new doc comments follow the repository's comment conventions? Check the commit-message and summary claims against the diff."
---

Scope is `internal/shuttleengine` only; the notice text, creel delivery and engine parsing are out of scope per the decision record.
