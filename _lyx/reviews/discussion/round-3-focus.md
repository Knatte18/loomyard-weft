---
round: 3
focus:
  - "Interim-notice premise and bound in `_lyx/discussion/decision-record.md` (the Decision that routes a fork's interim completion notice): does it settle whether a fork that sent an interim notice still counts as outstanding in shuttle (`internal/shuttleengine/claudeengine/background.go` matching rule and payload `status`), and does its Bound state what shuttle does for a non-waiting turn end with output files missing (`pollEventsTick` and `Wait` in `internal/shuttleengine/wait.go` hold it; `notifyHeld` in `hold.go` sends the parent a hold notice; the run ends at `master_timeout_min` or the step idle kill)?"
  - "Fixer-fork guard in the same record (the Decision's fixer-fork step that ends Merriam's turn on an interim notice): with `outcome_path` and `summary_path` already on disk, is a plain Stop by Merriam a gated `OutcomeDone` that `Wait` hands to `handleGatedBoundary` and the verify gate, regardless of whether Merriam rewrites the files, and does the Decision state the outcome of that arrival (a verify attempt spent on a dirty tree, a second fixer) as an accepted bound or a remedy?"
  - "Held-call activity reading in the record's D9 Bound: can a transcript whose newest entry is a `tool_use` with no result be told apart from an abandoned or stopped call, and what ends the active reading for a fork whose writer is gone, given loom's `step_idle_timeout_min` kill and batten's `notice_quiet_min` notice depend on it?"
---

The entries restate the sites they depend on and name no prior review.
