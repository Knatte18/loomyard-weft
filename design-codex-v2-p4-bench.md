# Bench Codex review block v2 on P4: one high run against two medium

`codex-v2-p4-bench` · Task · research, codex, burler · running · Darn-Write

## Question

On plan `P4` (machine-handover), does reading block v2 with a raised fork tool-call cap let Codex Astra's forked lens fan find the Claude fan's findings it has missed so far, and is one high-effort run or the union of two medium-effort runs the better buy?
Block v1 (`docs/research/codex-astra-tuned-prompt-bench-2026-10-27/data/prompts/tuned-block.md`) reached 3 to 4 of the Claude fan's findings per plan; no run found a finding that needs every reader of a changed symbol, or a read of code beyond the cards.
Its fork prompt asks the forks to read callers and code paths, yet keeps Claude's cap of 6 tool calls per fork, and the forks stopped at about 6 commands each.
The sibling tasks `codex-v2-p1-bench`, `codex-v2-p2-bench` and `codex-v2-p4-bench` run the same setup on one plan each; every condition but the plan is the same in all three.

## Cap

- One input: `P4`, with its snapshot from `docs/research/codex-review-bench-2026-10-25/data/inputs.txt` and the Claude fan reference recorded for it.
- Three forked fan runs of `gpt-6-astra`, each one `codex exec` handler spawning 3 lens forks with `fork_turns="all"`: one at high effort and two at medium effort, identical but for effort; never `xhigh` or above.
- No Claude run, no other repeat.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Method

- Reuse the tuned-prompt bench's harness under its `data/harness/` and its context: the PATTERN overview inlined as production renders it, the snapshot's `CLAUDE.md` as `AGENTS.md`.
- Use the lenses the Claude fan's round 1 on `P4` used.
- Replace block v1 with block v2 below, word for word, in the handler's prompt and the fork boilerplate, and keep it in `data/prompts/block-v2.md`.
- In the fork boilerplate change only the cap sentence's number, from `at most 6 tool calls` to `at most 20 tool calls`; keep the rest of the boilerplate as the tuned-prompt bench has it.
- Confirm from each rollout that the forks started, ran concurrently and inherited the handler's context, and that `AGENTS.md` was loaded; if not, stop and report.
- Record per run the wall time, the handler's and each fork's fresh input, output and reasoning tokens, each fork's own command count, and the windows; sibling benches run on the same account at the same time, so say so beside the window figures.

Block v2:

> ## How to read for this review
> 
> - Check every claim a card makes about existing code by reading that code and its callers, and read the code paths the change reaches beyond the cards.
> - For every symbol, type, config key, file or command a card changes, renames or removes, search the whole tree for every reader of it, with `grep -rn` or `lym quarry`, list the readers, and check that a card covers each one; a reader no card covers is a finding.
> - For every function a card changes, read the functions it calls and the code that calls it, one hop beyond the cards, and check that the change still holds there.
> - Where several cards change the same file or limit, check what each card's gate sees given the cards before it.
> - Read the `pattern/` background file of every PATTERN entry the plan touches, and check that each new command, package or file the plan adds appears where that entry requires.
> - Read the tests the plan adds or changes for waits, sleeps and timing that would slow or flake the suite.
> - Put several searches into one command where you can.

## Grading

Grade every run's handler findings, and any fork finding a handler dropped, against the Claude fan reference of `P4` as the earlier benches did: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Say for each of the Claude fan's `BLOCKING` findings on `P4` which run found it.
Compare three settings: the high run, each medium run alone, and the union of the two medium runs (distinct findings found by either), each with its findings, Claude fan findings found, `BLOCKING` found, unique real, false, fresh input tokens, window points and wall time, where the two medium runs count as run in parallel.
Report how far the two medium runs differ from each other, the first repeat measure of these benches.
Report beside every earlier Astra and Claude row for `P4`.

## Output

`docs/research/codex-v2-p4-bench-<date>/report.md` with its `data/`, landed as the run's commit.
