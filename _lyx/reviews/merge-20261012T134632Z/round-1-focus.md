---
round: 1
focus:
  - "internal/shedrun/paths.go (parent) against internal/shedrun/seed.go and the shedrun/shedengine tests (task): does the parent's new path helper agree with the task's reworked seed and state persistence, and does `internal/shedrun/doc.go` still describe both?"
  - "internal/websterengine/doc.go and the task's verifygate changes against the parent's websterengine test and doc edits: do both sides' edits to the doc and templates still describe the merged behavior?"
  - "internal/loomcli (approve.go, reject.go, wiring.go, landingdeps.go, peers.go) against the task's state package changes (internal/state/keep.go, state.go): does any parent loomcli call site rely on a state API or record shape the task changed or tolerates differently?"
  - "pattern/PATTERN-tolerant-records.md (task) against the parent's new records and stores (internal/parentreview/store.go, gate.go, loomengine/config.go): do the parent's new record readers follow the tolerant-records rules?"
  - "cmd/lyx/testdata/test-budget.yaml: do the merged top-level test counts per package fit the rows both sides edited, and did the merge make any test redundant with another?"
  - "internal/proc (task) against the parent's new process or wakeup code: does anything the parent added poll or probe processes in a way pattern/PATTERN-daemon-wakeups.md now covers?"
---

The overlap list is long but the interaction surface is narrow.
The task's changes concentrate in `state`, `proc`, `shedrun`, `shedengine` and `websterengine`.
The parent's concentrate in `loomcli`, `parentreview` and `loomengine`.
Entries therefore target the seams where one side's API or pattern is consumed by the other.
