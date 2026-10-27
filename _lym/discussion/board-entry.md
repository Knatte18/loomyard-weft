# Board entry `codex-astra-context-parity-bench`: Bench the forked Codex Astra fan with the Claude reviewers context

Forked Astra fan, high, on P4, with the PATTERN overview inlined and CLAUDE.md as AGENTS.md

## Question

Does Codex Astra's forked lens fan find the Claude fan's findings on a plan review once it starts from the same context the Claude reviewers had?
Every earlier Codex bench left out context the Claude reviewers had: the PATTERN overview a production reviewer gets inlined in its prompt (`docs/research/codex-review-bench-2026-10-25/report.md`), and the repository's `CLAUDE.md` and plugin skills a Claude session loads.
One of the Claude fan's three `BLOCKING` findings on `P4` is a PATTERN finding: `adoptcli` missing from `PATTERN-cli-cobra`'s background.
The forked fan at high effort, `docs/research/codex-astra-fork-fan-bench-2026-10-27/report.md`, found 1 of the Claude fan's 11 findings and none of its 3 `BLOCKING` ones.

## Cap

- One input: `P4`, the `machine-handover` plan, with the forked fan bench's snapshot, lenses and grading.
- One model and effort: `gpt-6-astra` at high effort for the handler and its forks; never `xhigh` or above.
- One forked fan run: one `codex exec` handler spawning 3 lens forks with `fork_turns="all"`, no repeat, no Claude run.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Method

- Start from the forked fan bench's harness under its `data/harness/` and change only the context.
- Give the handler the same context the Claude fan's round 1 on `P4` had: its prompt with the PATTERN overview inlined as production renders it, and the snapshot's `CLAUDE.md` in the snapshot's root as `AGENTS.md`, so Codex loads it as a Claude session loads `CLAUDE.md`.
  Record in the report every remaining difference from the Claude round's context, such as plugin skills Codex cannot load.
- Confirm from the rollout, as the forked fan bench did, that the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` was loaded; if either fails, stop and report.
- Record the wall time, the handler's and each fork's fresh input and output tokens, and the windows.

## Grading

Grade the handler's findings, and any fork finding the handler dropped, against the fan reference of `P4` as the earlier benches did: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Report beside the forked fan bench's row and the Claude fan row, and say for each of the Claude fan's three `BLOCKING` findings whether this run found it.

## Output

`docs/research/codex-astra-context-parity-bench-<date>/report.md` with its `data/`, landed as the run's commit.
