# Study what the Claude lens fan misses that Codex finds, and how to tune it

`claude-review-gaps` · Task · research, burler, stencil, claude · running · Drift-Verify

## Question

What kinds of real defects does the Claude lens fan miss that Codex Astra finds, and what changes to the Claude review stencils would catch them?
Across the Astra benches of 2026-10-27 on `P1` and `P4` (`docs/research/codex-astra-*-2026-10-27/`), every Astra run found real defects the Claude fan's round 1 lacks, 3 to 8 per run, several of them `BLOCKING`, and no false one.
The goal is the best review from both engines (operator, 2026-10-27); this task is the Claude half, read from data that already exists.

## Cap

- No reviewer run of any model: read only the existing reports, their `data/match.tsv`, the reviews under their `data/reviews/`, the Claude fan's round-1 reviews and lens outputs for `P1` and `P4`, and the round's stencils at the code commits in `docs/research/codex-review-bench-2026-10-25/data/inputs.txt`.
- One writer pass; no bench, no stencil edit.

## Method

- Collect every finding graded `unique-real` in any Astra bench of 2026-10-27, deduplicated per plan.
- For each, say why the Claude fan's round missed it: which lens owns its kind, whether that lens's text asks for the reading that finds it, whether the fork's tool-call cap or the handler's reading manifest left it out of reach, or whether no lens owns it.
- Group the misses by cause, and for each group propose one concrete change to the Claude review stencils (a lens's text, the fork boilerplate, the handler's holistic review or the reading manifest), worded as the text to add or change, and the findings it would have caught.
- Say for each proposal whether the same change would also suit the Codex prompt, so the two engines' prompts converge where they can.

## Output

`docs/research/claude-review-gaps-<date>/report.md`, landed as the run's commit; a bench of the proposals is a later task the operator decides.
