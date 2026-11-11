---
round: 2
ledger:
  - key: ended-early-causes-untested
    rounds: [1]
    status: resolved
  - key: darn-write-stop-on-fail-unpinned
    rounds: [1]
    status: resolved
  - key: budget-wiring-unpinned
    rounds: [1]
    status: resolved
  - key: orphaned-doc-comment-gateendedearly
    rounds: [1]
    status: resolved
  - key: untested-failure-branches
    rounds: [1]
    status: resolved
  - key: budget-notice-latest-failure-only
    rounds: [1]
    status: resolved
---

Round 1 raised six findings; round 2 raised none, so every key is resolved.
The round-2 review checked each in turn:

- `ended-early-causes-untested`: every `EndedEarly` cause is asserted in `internal/shuttleengine/gate_test.go`.
- `darn-write-stop-on-fail-unpinned`: `TestWire_DarnRecipe` pins `--stop-on-fail` on `darn-write` only.
- `budget-wiring-unpinned`: the budget wiring is pinned in `TestWire_DarnRecipe` and `TestVerifyGate_NoticesTheParentOnceItsBudgetIsSpent`.
- `orphaned-doc-comment-gateendedearly`: `gateEndedEarly` and `finalizeAndRemoveDiedWindow` each carry their own doc comment in `internal/shuttleengine/wait.go`.
- `untested-failure-branches`: the rerun and stop-on-fail branches are covered.
- `budget-notice-latest-failure-only`: `verifyBudgetNotice` names every failed run to date, latest first.
