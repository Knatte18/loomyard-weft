---
round: 5
focus:
  - "D6's venue for the halted child's way forward: decide whether the decision record names a surface the orch actually reads. `lyx batten status` adds `stuck_reason` only when batten's own state is `blocked`, and a halted child is a budget-exempt Stuck on the Run-Shed row that does not reach that state, so check the record's claim against `battenStatusExtras` in `internal/battencli/arm.go` and the persisted shed state."
  - "The awaiting child's retry hint versus D7's refusals: decide whether D6, D7 and the Acceptance rows agree on which verb each state's hint names. D7 refuses an awaiting run and names `lyx loom approve` or `lyx loom reject`; the awaiting hint in `internal/battenshed/innerrun.go` tells the reader to run `lyx loom start`; the halted reason is meant to name `lyx loom resume`."
  - "D7's dead-driver-strand refusal: decide whether the way forward it names works in every case it can arise. `lyx batten run <slug>` is refused while a batten run holds the run lock, and a live batten run revives a dead driver strand itself once per episode; `lyx loom start --no-attach` spawns a driver when none exists. Check the refusal and its Acceptance row name the verb that fits each condition."
  - "D7's driver-liveness read: decide whether the record says how a live driver strand that reed marked retiring is treated, given `resolveDriverStrandAction` in `internal/loomcli/bootstrap.go` splits live from live-but-retiring and `lyx loom start` never adopts a retiring strand."
  - "D3's verify deadline: decide whether the record uses one term for the parameter and the expiry, and states where the single production constant is declared, since the plan writer reads only the decision record."
---

Every entry restates the code sites it depends on.
`docs/overview.md` verb-list and batten-paragraph coverage in D8 is a scope question the reviewer may settle against CLAUDE.md's docs-in-the-same-commit rule.
