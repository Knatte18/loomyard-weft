---
round: 1
focus:
  - "`cmd/lyx/testdata/test-budget.yaml`, `contracts/stencils` row: both sides edited this file, and the task added top-level tests in `contracts/stencils/govalues_test.go` and `rubric_test.go`. Does the merged row still cover the package's top-level test count, and does `internal/languageconfig` still match the tests both sides added?"
  - "`contracts/stencils/markers_test.go` against the task's stencil edits: the parent added `repository_rules` to `orch-template-role`'s markers, and the task changed other stencils' markers. Does any merged stencil's marker set differ from what its test row and its filler in `internal/orchengine/prompt.go` and the task's `verify_tiers` fills supply?"
  - "`contracts/stencils/bouncer/bouncer-template-escalation.md` and the two `bouncer-fragment-decide-*` fragments (parent) against the task's rework, plan and Plan-Review wording (D1, D4): do the parent's new escalation and decide paths still agree with the task's rule that a Decision-overturning finding goes to the parent through `lyx loom decision add`, and with the late-scope route?"
  - "`contracts/stencils/orch/orch-template-role.md` (parent) against the task's D2 driver wording: does the orch role's new text still say who fixes a halted run and which verbs the parent runs, as the driver stencil's `## Ways forward` now states?"
  - "Test economy: does a test the parent added in `contracts/stencils` or `internal/orchcli` duplicate one the task added in `govalues_test.go` or `rubric_test.go`, and does any interaction behavior, such as a `verify_tiers` fill reaching a stencil the parent also changed, have no covering test?"
---

The overlap is one file and one package; the first entry checks the shared budget file, the others check where the parent's stencil edits meet the task's wording decisions.
