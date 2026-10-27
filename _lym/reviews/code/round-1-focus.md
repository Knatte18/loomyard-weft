---
round: 1
focus:
  - "internal/driftcli/wiring.go `verbStatus`: after the merge, does it still read status through the parent's `state.ReadJSON` and carry no `AwaitRow` or seed read, so `lym drift` verbs behave the same for loom and darn runs?"
  - "internal/loomcli/driftwiring.go and internal/loomcli/wiring.go: do the parent's `ReadJSON` call sites sit beside the task's removal of the `awaitRow` parameter and `Deps.AwaitRow` without a leftover reference or a stale comment on the changed lines?"
  - "Does anything the parent added since the settle (callers, tests, docs) still use `AwaitRow`, `WaitCapture` or the `capture` config key, so the removal the board entry asks for is incomplete after the merge?"
---

The amendment note names three files the parent and the task both changed.
The parent's change to them is the `ReadJSONNoMkdir` to `ReadJSON` swap, which is independent of the task's change.
The focus checks that the merge keeps the task's removals intact; the settled remainder is out of scope.
