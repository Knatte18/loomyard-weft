---
round: 1
focus:
  - "The parent's `contracts/specs/refusal-spec.md` change against the task's edited stencils and rubrics under `contracts/stencils/`: does any stencil or rubric wording the task added contradict or paraphrase the changed spec text, given `PATTERN-producer-pointer-rule`?"
  - "The parent's `PATTERN.md` and `pattern/PATTERN-model-choice-mapping.md` changes against the task's `CLAUDE.md` and `docs/overview.md` edits: does any task wording about the board lock, the documentation lifecycle or `PATTERN-documentation-lifecycle` now disagree with a changed or added PATTERN entry?"
  - "Whether any parent-side stencil, rubric, `CLAUDE.md` or `docs/overview.md` change (check `git diff` of the parent range for those paths) touches the same rule or sentence as the task's edits and merged without a textual conflict."
  - "Whether the parent's test-budget or enforcement-test changes (`cmd/lyx/testdata/test-budget.yaml`, stencil-scanning tests) cover or constrain the task's new stencil prose, such as a test that scans stencils for forbidden phrases or required includes."
---

The task's range touches only docs, stencils and rubrics; the parent's touches Go code, specs and pattern files, with no overlapping files.
The interaction is therefore mostly textual: stencil and doc wording against the parent's new spec and pattern text.
