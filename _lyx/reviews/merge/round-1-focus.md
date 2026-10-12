---
round: 1
focus:
  - "Shared-file and cross-package seams: the parent range (planparser overlay, planglyph, loomcli validate/overlay, websterengine state) and the task range (boardengine store/task/migrate, lyxtemp sweep, boardcli) share no file, so check whether any parent code reads board or temp-sweep behavior the task changed, such as `websterengine` state or `loomcli` paths that touch `boardengine` task fields, `lyxtemp` roots or the run lock."
  - "`cmd/lyx/testdata/test-budget.yaml`: if both sides edited it, settle whether the merged rows still match the packages' top-level test counts, and whether any package the task's new tests pushed over its row is still covered after the parent's edit."
  - "`pattern/PATTERN-gate-self-check-parity.md` gained the `validate-plan --overlay` line: settle whether any task code adds or changes a gate or self-check verb that this file's parity rule now requires to be listed or to share a function."
  - "Test economy: settle whether the task's new `boardengine` and `lyxtemp` tests duplicate coverage the parent's new tests give, and whether the interaction of the parent's overlay or `websterengine` state changes with the task's board-field handling has any covering test."
---

Seed focus for the merge perch.
The two ranges share no file or package, so the interaction is by call path and shared contracts, not textual overlap.
