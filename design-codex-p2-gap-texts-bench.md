# Bench Codex on P2 with block v2 and the gap study lens texts, two medium runs

`codex-p2-gap-texts-bench` · Task · research, codex, burler · done

Bench the forked Codex Astra fan on plan `P2` with block v2 plus three of the gap study's lens and boilerplate texts, at medium effort, and measure real findings per point of the weekly window.

## Why

On `P2` (`quarry-csharp`) block v2 was weakest: each run found 1 or 2 of the Claude fan's 13 findings, the forks ran only 2 to 7 commands each, and the `way-forward` fork returned no finding in any run (`docs/research/codex-v2-p2-bench-2026-10-27/report.md`).
`docs/research/claude-review-gaps-2026-10-27/report.md` proposes lens and boilerplate texts for those gaps; this bench tries three of them on Codex.
The Codex weekly budget is getting used up by benching (operator, 2026-10-27), so the bench is small and its cost is measured exactly.

## Setup

Reuse `codex-v2-p2-bench`'s harness, inputs, context and prompts (`docs/research/codex-v2-p2-bench-2026-10-27/data/`) unchanged except for these three texts, each copied word for word from the gap study:
- Proposal 1: the new last sentences of lens `guard-reach`, and its reading bullet added to the fork-reading block.
- Proposal 2: the added `guard-reach` sentence, and "one hop beyond it" changed to "beyond it along the input's path" in the fork boilerplate.
- Proposal 4: the added `way-forward` sentences.
Context parity as before: the PATTERN overview inlined and the snapshot's `CLAUDE.md` as `AGENTS.md`; fork cap 20.

## Runs, exact cap

- Exactly 2 runs, both `gpt-6-astra` at `medium` effort, each a forked fan (`spawn_agent`, `fork_turns="all"`, three lens forks), on `P2` only.
- No high run, no other model, no other plan, no repeat beyond these 2.
- Run the two one after the other, with no other Codex bench on the account at the same time, so each run's window points are its own: read the weekly `used_percent` before and after each run.

## Grading and report

`docs/research/codex-p2-gap-texts-bench-2026-10-27/report.md`.
The main measure is real findings, not overlap with the Claude fan: two runs of a reviewer at this temperature do not find the same things, so matching one Claude round's list is a weak measure (operator, 2026-10-27).
- Per run: findings, real, `BLOCKING` real, false, the Claude fan's `BLOCKING` findings found, commands per fork, findings from the `way-forward` fork, fresh input tokens, window points, wall time.
- Real findings and real `BLOCKING` findings per window point, against the two medium runs of `codex-v2-p2-bench`.
- One grader; one input; state that a difference of one or two findings is inside the run-to-run spread.
