---
round: 1
focus:
  - "internal/partsparser/check.go and check_test.go: the new refusal of an export that names an existing member or file without a `plan:` handle. Settle whether the existing-member test matches the part gate's own `part-export-missing` rule, and whether a legitimate export of a new name, or a plan: handle, can be refused by mistake."
  - "internal/partsparser/check.go, internal/planparser/exportname.go, vocabname.go, validate.go, join.go: the Parts-Write gate now runs `handle-malformed` and `fabric-vocabulary-name` on each `plan:` export. Settle whether the checks are shared with `validate-plan --part` or duplicated, whether the two paths can disagree, and whether join.go lost a check in the refactor."
  - "ExportMet change: an export now counts only what a part's cards create, not a re-sign or delete. Settle whether every caller (partcheck.go in loomshed, validate.go in loomcli, planparser join) still reaches the verdict the decision record's Goal asks for, and whether the changed tests still cover the case that motivated it."
  - "contracts/stencils/loom/loom-template-parts.md and loom-template-part-writer.md: read against the Goal's first change, that Parts-Write reads the PATTERN entries for every package a part exports from before it names an export. Settle whether the new wording asks for that, and whether it drops, weakens or contradicts a rule the Goal did not ask to change."
  - "contracts/specs/loom-parts-spec.md and loom-plan-checks-spec.md, internal/loomcli/doc.go, internal/partsparser/doc.go, internal/shedrecipe/entries_gate.go: settle whether the spec and doc comments state the same refusal rules the code now enforces, and whether the entries gate change is licensed by the Goal."
  - "docs/research/plan-split-tune-2026-11-02.md against the parent's Added after Discussion entry: settle whether it records the round-1 changes and their reasons, marks the replay results pending the `plan-split-replay` follow-up, and claims nothing the diff does not bear out."
  - "Scope: the diff must make no replay, scratch worktree or later-round change. Settle whether any changed file goes beyond the Goal's changes and the bench report's named causes of a hand edit."
  - "Test economy in check_test.go, exportname_test.go, partcheck_test.go, join_test.go, entries_gate_test.go: settle whether each new test covers behavior no existing test covers, and whether any added behavior has no test."
---

Seed focus for round 1.
The answer key is the decision record alone: the Goal, amended by the parent's 2026-11-02 entry that ends this run after round 1's tuning with no replay.
