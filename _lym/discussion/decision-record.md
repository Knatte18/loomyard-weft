## Goal

Answer, from existing data, what kinds of real defects the Claude lens fan misses that Codex Astra finds on the plans `P1` and `P4`, and which changes to the Claude review stencils would catch them.
The deliverable is one report, `docs/research/claude-review-gaps-2026-10-27/report.md`.

## Scope

- In: a desk study of the Astra benches of 2026-10-27 under `docs/research/codex-astra-*-2026-10-27/`, the Claude fan's round-1 plan reviews of `P1` and `P4` from the weft history, and the review-plan stencils at HEAD and at the two code commits in `docs/research/codex-review-bench-2026-10-25/data/inputs.txt`.
- In: one report with, per cause group, one proposed stencil change worded as text to add or change, the findings it would have caught, and whether it also suits the Codex prompt.
- Out: any reviewer run of any model, any bench, any stencil edit, any change to the board entry.

## Decisions

### Population of findings

Decision: the population is every row labelled `unique-real` in the `data/match.tsv` of the eight Astra benches dated 2026-10-27, deduplicated per plan by defect, a defect found in several benches counting once.
Rationale: the board entry names those benches; the earlier solo bench of 2026-10-25 is outside the named set.

### Cause evidence

Decision: a miss is attributed from the lens text, the fork boilerplate, the reading-manifest block and the plan rubric, and from what the Claude round-1 review accepted or rejected in its own text; no claim is made about what a Claude fork read.
Rationale: the Claude forks' reports and the handler's manifest are not kept in the weft history for these two rounds, so a statement about their reading would be a guess.

### Cause groups

Decision: misses are grouped by the reading or reasoning step that finds them, not by lens, and each group gets exactly one proposal.
Rationale: the task asks for one concrete change per group, and one lens can own several groups.

### Proposal form

Decision: each proposal names the stencil file or lens it changes, gives the text to add or change, lists the findings it would have caught, and states whether the Codex prompt takes the same text.
Bound: a proposal is unmeasured; the report says no proposal is shown to catch its findings until a later bench runs it.
Rationale: the cap forbids running a reviewer, so a proposal is a hypothesis tied to named findings.

### Convergence

Decision: a proposal that edits a lens text or the fork reading block is marked as shared with Codex when the Astra tuned block or the Astra fork boilerplate can carry the same sentence unchanged.
Rationale: the operator's goal is the best review from both engines, with prompts that converge where they can.

## Constraints

- No file other than the report is written outside `_lym/`.
- `docs/research` reports state measurements and name sources; a number is cited from a file or derived by a shown command.
- Markdown keeps one sentence per line.

## Auto-mode assumptions

- A darn run has no operator; the grouping of misses into causes is the writer's judgment, stated in the report as such.
- Where a finding is arguable in its bench grading, the report keeps the grading's own word.

## Open risks

- One plan pair and one run per cell: a group built on one or two findings is a hypothesis.
- The Claude lens outputs of these two rounds are not preserved, so a group whose cause is a fork's reading depth is an inference.

## Acceptance criteria

- The report lists every deduplicated `unique-real` finding of the eight benches with plan, severity and the benches that found it.
- Each finding is assigned to exactly one cause group, and the group states which lens owns its kind and whether that lens text asks for the reading.
- Each group has one proposal worded as text, with the findings it would have caught and the Codex verdict.
- The report states the evidence limits and that no bench or stencil edit was made.
