---
round: 1
focus:
  - "internal/shuttleengine/run.go: the parent's `Run.SessionID` sits beside the task's `InterruptRestoring` and `interruptEngine`; settle whether the merged file compiles into one coherent API with no duplicated or shadowed helper, and whether either side assumes the other's state fields are unchanged."
  - "internal/orchengine/doc.go: the task reworded the idle-probe residual (`non-empty box other than the provider's placeholder`) while the parent added the session-id record, `--continue` section and residuals around it; settle whether the merged doc still describes one consistent idle probe and cycle behavior, and whether the parent's claim that the watcher reads the session each tick holds with the task's box-read changes."
  - "internal/orchengine watcher idle probe versus the task's placeholder-blank and box-read changes in shuttleengine and claudeengine: settle whether the parent's per-tick session read and strand-checked save interact correctly with the task's changed box reading."
  - "cmd/lyx/testdata/test-budget.yaml: settle whether each package's merged row matches the merged tree's top-level test count for `internal/orchengine`, `internal/shuttleengine` and `internal/shuttleengine/claudeengine`, since both sides raised rows independently."
  - "internal/shuttleengine/run_test.go: settle whether the parent's `TestReadSessionStates` session-id rows and the task's new interrupt tests merged without redundant coverage, and whether the parent's `writeReadRun` default record id affects any task test."
  - "Coverage of the interaction: `internal/orchcli/start.go` takes `run.SessionID()` while `internal/creelcli/delivery.go` uses `InterruptRestoring`; settle whether any behavior where orch's recorded session id meets an interrupt or restored-prompt clear on the same strand is uncovered."
---

Seed focus drawn from the four overlapping files and the parent's note that its `shuttleengine` change is additive.
