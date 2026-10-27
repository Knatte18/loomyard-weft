# Study what the Claude lens fan misses that Codex Astra finds

Adds `docs/research/claude-review-gaps-2026-10-27/report.md`, a desk study of the Astra benches of 2026-10-27 on the plans `P1` and `P4`.
It collects the 28 distinct `unique-real` defects those benches found, groups them into six causes, and for each group proposes one change to the Claude review-plan stencils, worded as the text to add.
Nothing else changes: no reviewer ran, no bench was built and no stencil was edited.

## What the report holds

- A table of the 28 defects (18 on `P4`, 10 on `P1`; 9 `BLOCKING`, 15 `MEDIUM`, 4 `LOW`) with plan, severity, the benches that found each, and its group.
- Per group: the lens that owns the kind, whether that lens's text asks for the reading that finds it, why the Claude round-1 review missed it, one proposal as text for a named file, the findings it targets, and whether the Codex prompt takes the same text.
- Proposals 1 to 4 edit the `guard-reach`, `affected-set` and `way-forward` lens texts in `internal/lenses/template.yaml` and two bullets of `contracts/stencils/burler/burler-fork-reading.md` and the fork boilerplate in `burler-cluster-fan.md`; proposals 5 and 6 add a sentence each to the handler's holistic paragraph in `burler-cluster-fan.md`.
- Cross-cutting observations on the tool-call cap, the focus directive and the plan the review receives, and a list of what the study does not show.

## Design decisions

- **Population.** A defect counts when any of the eight `codex-astra-*-2026-10-27` benches labels it `unique-real`, once per plan under its highest bench severity. The earlier solo bench of 2026-10-25 is left out as outside the benches the board entry names.
- **Cause evidence.** The Claude forks' reports and the handler's reading manifest are not in the weft history for these two rounds. Causes are therefore read from the lens text, the fork boilerplate, the reading block, the plan rubric and what the Claude review text accepted or rejected. The report states that it makes no claim about what a Claude fork read.
- **Groups by reading step, not by lens.** One lens owns several unrelated reading steps, so a per-lens proposal would mix causes.
- **No new lens.** 24 of the 28 defects were reported by an Astra fork under one of the three existing lens names; the report proposes sharper procedures for those lenses.
- **Shared rejection counted.** `P4-09` (the `seamsignature_test.go` list) was rejected by the Claude review and called marginal by the Astra handler; it stays in the population because Astra reported it, and the report says it is not a miss.
- **Arguable gradings kept.** Where a bench marks a finding arguable (`P4-13`, `P4-15`, `P1-05`, `P1-06`), the report keeps that word.

## Verification

Documentation only: the change adds one markdown file, so no build or test reads it and `lym gate test` has nothing to run.
The decision record and support log pass `lym loom validate-discussion`.
