---
round: 1
focus:
  - "internal/loomcli/cli.go resolvePersistentPreRun (parent skips the `checklist` verb by name) against the task's parent-review/alignment verbs added to loomcli: does any task verb or group share the name `checklist`, or lose its arming through the name match?"
  - "internal/loomcli/bootstrap.go, validate.go, start.go and the deleted statusbuild.go (parent) against the task's alignment wiring and its callers: does any task code still call a removed or reshaped symbol, or assume the old bootstrap/validate shape?"
  - "internal/shedrun/paths.go NoticeLog/NoticeLogRel (parent) against the task's shedrun additions and paths_test.go: do the two sides' path constructors, tests and the envkit change agree on `.lyx` hub-level files, and does any task sweep or lock path now collide with `notices.log`?"
  - "pattern/PATTERN-daemon-wakeups.md (statuspane loop added to the unbounded-wait list) against the task's new waits, such as the parent-review wait (`parent_review_wait_min`): does any task loop that lives as long as a run follow the backoff and event-source rule?"
  - "pattern/PATTERN-gate-self-check-parity.md (`validate-plan --overlay`) against the task's gate or self-check verbs: does the task add or change a gate or verb whose parity entry is now missing or contradicted?"
  - "internal/loomrecipe names.go RowNames and recipe_test.go, contracts/stencils (stencils.go, claims_test.go) and internal/loomshed/gates.go against the task's recipe or stencil edits: do row-name, claim and stencil-registration tests still cover the task's new rows and stencils, or has one side's edit left the other's coverage unreachable?"
  - "cmd/lyx/testdata/test-budget.yaml, docs/overview.md, internal/loomcli/doc.go and contracts/specs/refusal-spec.md: did both sides edit the same rows or sections so that the merged text drops, double-counts or contradicts either side's entry?"
  - "Test economy across internal/loomcli and internal/battencli: do the parent's new overlay/checklist/overview tests and the task's alignment tests cover the same behavior twice, or leave a behavior of the interaction untested?"
---

Seeded from the overlap list and the parent's diff.
Each entry names a place where both sides touched shared code or a shared invariant; none pre-states a verdict.
