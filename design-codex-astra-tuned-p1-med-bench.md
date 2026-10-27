# Bench the tuned forked Codex Astra fan on P1 at medium effort

`codex-astra-tuned-p1-med-bench` · Task · research, codex, burler · running · Darn-Write

## Question

How does Codex Astra's forked lens fan with the Claude reviewers' context and the tuned reading block do on `P1` at medium effort?
`docs/research/codex-astra-tuned-prompt-bench-2026-10-27/report.md` ran that setup on `P4` at high effort: 4 of the Claude fan's 11 findings and 2 of its 3 `BLOCKING` ones, none false.
This run keeps every condition of that bench and changes only what the cap names.

## Cap

- One input: `P1`, the `gate-runs-once` plan, with the earlier benches' snapshot of it from `docs/research/codex-astra-solo-bench-2026-10-25/data/inputs.txt` and the Claude fan reference recorded for it.
- One model and effort: `gpt-6-astra` at medium effort for the handler and its forks; never `xhigh` or above.
- One forked fan run: one `codex exec` handler spawning 3 lens forks with `fork_turns="all"`, no repeat, no Claude run.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Method

- Reuse the tuned-prompt bench's harness under its `data/harness/`, its context (the PATTERN overview inlined as production renders it, the snapshot's `CLAUDE.md` as `AGENTS.md`) and its reading block `data/prompts/tuned-block.md` unchanged, word for word.
- Use the lenses the Claude fan's round 1 on `P1` used.
- Confirm from the rollout that the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` was loaded; if either fails, stop and report.
- Record the wall time, the handler's and each fork's fresh input and output tokens, and the windows; sibling benches may run on the same account at the same time, so say so beside the window figures.

## Grading

Grade the handler's findings, and any fork finding the handler dropped, against the Claude fan reference of `P1` as the earlier benches did: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Say for each of the Claude fan's `BLOCKING` findings on `P1` whether this run found it.
Report beside the tuned-prompt bench's row and every earlier Astra and Claude row for `P1`.

## Output

`docs/research/codex-astra-tuned-p1-med-bench-<date>/report.md` with its `data/`, landed as the run's commit.
