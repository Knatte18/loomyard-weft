---
round: 1
focus:
  - "internal/shuttleengine: the parent dropped `render.AnchorBelowParent` use from `spec.go`, `attach.go` and the shuttleengine tests while the task added `AcceptStartupGate` and the `NoInputBox` probe in `run.go` and `session.go`. Settle whether the task's new code or its new tests (`startup_test.go`, `seam_enforcement_test.go`, `session_test.go`) still assume a below-parent or shell-window anchor, or the old default."
  - "internal/shuttleengine/doc.go and `pattern/PATTERN-daemon-wakeups.md`: the task edited both while the parent changed shuttleengine docs and comments around the anchor default. Settle whether the merged text contradicts itself or the merged code."
  - "cmd/lyx/testdata/test-budget.yaml: both sides edited it. Settle whether each package's row matches the merged top-level test count, with the task's `internal/shuttleengine` bump and the parent's other rows both kept."
  - "internal/creelengine and internal/testkit/creelfake against what the parent changed elsewhere (shuttle, reed or webster callers): settle whether a contract the task's creel changes call, such as the shuttleengine probe or accept-gate API, was altered by the parent."
  - "Test economy: settle whether a test the parent added or rewrote in shuttleengine now duplicates one of the task's new tests in `startup_test.go` or `session_test.go`, and whether the `AcceptStartupGate` and `NoInputBox` interaction has an uncovered behavior."
---

Seed pass only; each entry names a place and a question, with no verdict pre-stated.
Checked against `Do not flag`: each entry concerns the interaction of the two sides, not a defect in either side alone.
