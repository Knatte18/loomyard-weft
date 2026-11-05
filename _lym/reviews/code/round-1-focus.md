---
round: 1
focus:
  - "internal/beamcli/mergeparent.go: does the filed task match the Design — a darn recipe task in the beam's project, perch on, filed at any time, started by dobby like any member? Check the failure paths (unknown beam, no parent, filing refused)."
  - "internal/beamcli/land.go and land_test.go: the diff changes land; does it stay within the entry's scope, and does the protected-parent landing behavior from the base remain intact?"
  - "internal/reedengine/template_posix.yaml and template_windows.yaml: do both templates give the beam group `merge-in` and `land` with confirm prompts and the commands `lym beam merge-parent {beam}` and `lym beam land {beam}`, and does the `prime` group carry the beam group's other buttons without these two instead of falling back to the beam group?"
  - "contracts/stencils/loom/loom-rubric-code-review.md: read the four added lines against the entry; does the stencil change drop, weaken or contradict an existing rule, and does the entry ask for it at all (scope)?"
  - "Commit messages and summary: the entry says the merge task's perch reviews the merge alone and pawl neither starts nor requires such a run; does the diff touch pawl or add a goal check before landing that the entry excludes?"
  - "Test economy: mergeparent_test.go against the public surface of the command; does any test duplicate another, and does every added behavior (merge-parent filing, the prime group buttons) have coverage?"
---

Seed focus from the decision record's Goal, which is the only answer key.
The record's other sections are empty, so there is no plan or acceptance criteria to check.
