---
round: 3
focus:
  - "D5's handling of a cycle request made while a hub pause holds the cycle machine: follow the request through every clock `internal/orchengine/watcher.go` runs on it, `tickIdle`'s stale-request age and `tickHandoff`'s phase timeout started at `CycleRequestedAt`, and settle whether each excludes the paused time and whether an acceptance criterion requires the cycle to run to completion after the resume."
  - "D5's serialization of `cycle-request`: settle whether every writer and remover of the marker, `RequestCycle` in `internal/orchengine/state.go` and `requestCycle` in `internal/orchcli/refresh.go` with no live watcher included, holds the orch state lock, so a request written after a removal's read cannot be removed."
  - "D1 condition 2 and its acceptance criteria against `internal/reedengine/strand.go` `validateDisplay` and `internal/reedengine/ownwindow.go` `presentPanes`: settle that every term and input the record names, such as a shared-window strand, can exist in reed as built."
  - "D2's pane lookup against `internal/reedengine/reconcile.go`: settle whether the recorded pane id survives the reconciles that every public reed op, and `lyx orch start` before it reads strands, run, so the placement D2 reports is visible when its acceptance criterion says it is."
  - "D5's withdraw behavior against the operator's board-entry request that the verb refuse once the cycle has started: settle that the record either follows the request or lists the departure with its reason."
---

Focus targets the open design keys; the remaining findings are local corrections whose fixes the reviewer can verify without a pointer.
