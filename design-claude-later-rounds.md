# Desk study: which Astra-found defects the later Claude review rounds caught

`claude-later-rounds` · Task · research, burler · draft

Depends on: [`codex-v2-p4-bench`](design-codex-v2-p4-bench.md)

Desk study: of the defects Codex Astra found and the Claude lens fan's round 1 missed, which did the Claude plan-review loop's later rounds catch?

## Why

Every Astra bench of 2026-10-27, and the gap study `docs/research/claude-review-gaps-2026-10-27/report.md`, compared one Astra review with the Claude fan's round 1 only.
A loom plan review runs several rounds of the same reviewer, and no single round catches everything (operator, 2026-10-27).
The plans benched went further rounds: `P1` (`gate-key-shape`) has rounds 2 and 3, `P4` (`machine-handover`) has rounds 2 and 3, and `P2` has at least round 2, as plan review files in `loomyard-weft` on the commits after the round-1 commits the benches name.
The fair question is what the whole Claude loop misses, not what its first round misses.

## Scope

- No reviewer runs at all: no Codex run and no Claude run. The Codex weekly budget is getting used up (operator, 2026-10-27).
- Inputs: plans `P1`, `P2` and `P4`; the Claude plan reviews of every round after round 1 on each, and the plan's final approved state, all read from `loomyard-weft` history.
- Population: every `unique-real` defect in the `data/match.tsv` of the Astra benches under `docs/research/codex-astra-*-2026-10-27/` and of the reports the tasks `codex-v2-p1-bench`, `codex-v2-p2-bench` and `codex-v2-p4-bench` land under `docs/research/`, deduplicated per plan as the gap study does, reusing its ids `P1-nn` and `P4-nn` and adding `P2-nn`.

## Method

For each defect, read the Claude reviews of rounds 2 onward and the cards as they stood after the last fix, and grade it as one of:
- `caught-later`: a later Claude round names the same defect (give the round and finding id).
- `fixed-incidentally`: no finding names it, but a later card change removes it.
- `still-in-plan`: the approved plan still has it.
Also state, per plan, how many rounds ran and each later round's finding count and verdict.

## Report

`docs/research/claude-later-rounds-2026-10-27/report.md`:
- a table per defect with its id, severity, gap-study group (G1 to G6, or new) and grade;
- per plan and per group, counts of each grade, and the `BLOCKING` ones apart;
- what this changes in the gap study's six proposals: which groups the loop already closes and which stay open after the last round;
- what it means for a Codex reviewer's place in the loop (for example only round 1, or every round), stated as a hypothesis.
One grader; say so in the limits.
