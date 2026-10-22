---
round: 1
focus:
  - "internal/shuttleengine/doc.go: the task's gate-findings text and the parent's PermissionMode/bypass rewrite both edited this file; check the merged doc still reads true against `GateOutcome.Findings`/`FindingsPath` in gate.go and the parent's Spec/OpenSpec PermissionMode comments."
  - "docs/overview.md: both sides edited it; check the merged module table and execution stack are consistent with the task's darn classifier and the parent's changes."
  - "internal/loomcli: the task's darnclassify.go, darnwiring.go, landingdeps.go and wiring.go against the parent's decision.go additions (new decisionDeps seams, creelcli/shuttleengine imports); check no shared dependency struct, seam list or wiring test (wiring_seams_test.go) assumes the other side's shape."
  - "Gate findings kept past the run directory (task: `GateOutcome.Findings`, wait.go `evaluateGate`) against the parent's shuttleengine/claudeengine changes (bypass default PermissionMode); check no parent-side code path builds or reads a gate outcome or finishes a run in a way that skips the task's findings handoff."
  - "Parent-changed invariants in PATTERN.md or pattern/ files: none listed as changed; check only that the parent's decision.go notice flow and the task's darn routing do not both claim the same live-run or strand handling."
---

Seeded from the overlap note and the two sides' diffs of the overlapping files and packages.
