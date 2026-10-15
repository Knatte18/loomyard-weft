---
round: 1
focus:
  - "internal/battencli/drive.go: the parent moved the step and goto seams into driverStep and driverGoto (caller label shedengine.CallerShed, SettleHandBack now in shedrun) while the task changed this file too; settle whether the task's engine-registry edits to the driver deps, run record or shuttle spawn still line up with the new seams."
  - "internal/reedengine/doc.go: both sides edited the package doc; settle whether the task's registry description and the parent's FollowSession, @lyx_follow and @lyx_bindings_build passages still describe the merged code, with no dropped or duplicated paragraph."
  - "pattern/PATTERN-leaf-packages.md and pattern/PATTERN-pane-binary-resolution.md: the parent changed both (the new sleepjournal leaf, the pane binary resolution line); settle whether any package the task added or moved, such as an engine registry or a codex-ready seam, breaks the leaf rule or resolves a pane binary another way."
  - "internal/loomcli, internal/reedcli, internal/websterengine: packages both sides touched without a file overlap; settle whether the task's callers of shuttle engine selection compile and behave against the parent's changed signatures, in particular reed follow wiring and the websterengine template test."
  - "contracts/specs/refusal-spec.md and docs/overview.md: settle whether the merged text lists the refusals and the module table the task and the parent each added, with no entry lost in the merge."
  - "cmd/lyx/testdata/test-budget.yaml: settle whether the merged budget rows still fit the merged test counts of each package both sides added tests to, and name any test the merge made redundant with a covering test or any interaction behavior with no coverage."
---

Seed focus drawn from the overlap note and a skim of the merge diff.
The parent's range is large (new `sleepjournal`, `shedrun` settle, `reedengine` follow), so the interaction surface is the task's registry seam against those changed signatures and docs.
