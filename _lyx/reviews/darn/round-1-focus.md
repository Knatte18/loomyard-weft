---
round: 1
focus:
  - "`newAgentWindowSelected` in `internal/reedengine/ownwindow.go`: does the current-window test match the board scope, moving only a viewer on the previous newest agent window or the Driver window, and never one the operator moved by hand to an older agent window or the shell?"
  - "`relevantWindow` and the Driver-role match through `internal/agentname`: is the window listing taken before the new window opens, and does an own-window-only Driver check hold for every spawn path, including a strand joining a shared group window and a replaced agent?"
  - "`newAgentWindowTakesViewerLocked`: the added window listing and its failure path, which keeps the viewer and logs at Warn; check lock discipline and that no new tmux call races the spawn."
  - "Summary claim that replacing the viewed agent lands the viewer on the replacement, and the changed expectation in the `TestWindowOrder_RealTmux` replace subtest: does the diff bear it out, and does it stay inside the amended board scope?"
  - "Tests against `PATTERN-test-economy`: does a `TestWindowOrder_RealTmux` subtest duplicate a `TestNewAgentWindowSelected` table row, and is any added behavior uncovered? Is the `test-budget.yaml` bump the one the budget rule requires?"
  - "Docs in the same diff: the focus-rule paragraph in `internal/reedengine/doc.go` states the widened rule exactly, and any other doc or `PATTERN.md` text describing the old rule is updated."
---

Seed focus only; the base diff review and the rubric's `Also flag` dimensions still apply in full.
