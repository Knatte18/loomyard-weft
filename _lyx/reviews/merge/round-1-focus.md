---
round: 1
focus:
  - "internal/websterengine/doc.go, internal/webstercli/doc.go and internal/loomcli/doc.go: the parent added a cold Opus-high verify fixer and the task dropped retired PR-Gate, Merge-Bouncer, PR-Rework and reject-review text. Check whether the merged docs and comments still name any retired segment or gate, or omit the new fixer where they describe the verify path."
  - "internal/loomcli/decision.go and internal/loomcli/checklist_test.go: both sides changed them. Check whether the parent's decision-add change (replacing the PATTERN bullet for an already declared entry ID) and the task's wording and rework-check edits still agree, and whether the tests still pin the behavior the merged code has."
  - "contracts/stencils/stencils.go and contracts/stencils/markers_test.go: check that markers the task removed or renamed are not still expected by the parent's new or changed stencil text or tests, and that no stencil marker is left without a test or consumer."
  - "contracts/specs/loom-plan-spec.md and contracts/specs/llm-model-spec.md: check that the specs match both the task's removal of retired segments and the parent's new verify-fixer and model usage, with no spec text naming a recipe row, gate or model the merged code no longer has."
  - "internal/fabricengine/doc.go, internal/orchcli/doc.go and docs/overview.md: check the module table and execution stack against the parent's handoff-timeout, status overview and Bolt-commit changes and the task's removals; look for a doc claim that only one side's change makes false."
  - "internal/websterengine (verifyfixer.go, verifygate.go, runlevel.go, render.go, template.yaml) against the task's webster record comment and envkit publish gate list changes: check whether the task's edits assume a gate or segment list that the parent's fixer path extends."
  - "Test economy: for the overlapping packages, look for a test the merge made redundant with another, and for a merged-interaction behavior with no coverage, naming the test or behavior."
---

Seed focus from the overlap list and the parent's commits (verify fixer, decision add, handoff timeout, status overview, Bolt commit skips) against the task's retired-text cleanup commits.
