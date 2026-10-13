---
round: 1
focus:
  - "internal/shuttleengine/spec.go: both sides edited the Spec comments (the parent's ForkSubagents record line, the task's Display/Anchor default). Check the merged comments still read coherently and that the task's own-window default matches the parent's shuttleengine spawn, send and run changes (run.go, send.go, readiness.go) and the doc.go lines each side added."
  - "internal/shuttleengine/doc.go: the task removed the orch/loom-driver below-parent sentence; the parent added doc lines on turn progress and fork audit. Settle whether the merged doc still describes where a run's strand lands, and whether any parent-added text assumes the removed sentence."
  - "docs/overview.md: the reed, shuttle, orch and loom-start entries were edited by both sides (task: overview window, window order; parent: shuttle and plan-lens text). Check the merged entries contradict neither side and that the shuttle entry's strand-placement claims match the merged code."
  - "internal/loomcli/doc.go: the task rewrote the checklist-strand text (overview window, no content-size report); the parent added the validate-plan lens-review section. Check the merged package doc is consistent with the merged loomcli code."
  - "cmd/lyx/testdata/test-budget.yaml: both sides changed rows. Check each package's row covers the merged top-level test count (reedengine, selfexec, and the packages the parent added tests to), and that none of the task's reedengine tests are made redundant by parent changes."
  - "Parent's change to pattern/PATTERN-gate-self-check-parity.md (Plan-Write fork-authorized gate, ValidatePlanLensReview): check the task's code adds no plan-gate or validate-plan path that bypasses the new pairing."
  - "Window-order change versus parent's shuttle changes: check any caller of shuttleengine or reedengine placement APIs that the parent added or changed (spawn, gate, sessionsignal) still agrees with the task's removal of SetContentRows, InstallPins option and SkipWhenBoxEquals."
---

Overlap is almost all documentation and the test budget; code overlap is limited to the `Spec` comments.
The focus entries ask whether the merged docs and the two sides' code still agree.
