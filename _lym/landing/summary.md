# Bench the tuned forked Codex Astra fan on P1 at high effort

Adds `docs/research/codex-astra-tuned-p1-high-bench-2026-10-27/`, a report and its `data/` for one `codex exec` handler run on the plan `P1` (`gate-runs-once`) at `gpt-6-astra` high effort, spawning three lens forks with `fork_turns="all"`.
It repeats the tuned-prompt bench's setup on a second plan: the same harness, the inlined PATTERN overview, `AGENTS.md` and the reading block, with only the input changed.
No production code changes.

## Result

The handler returned `BLOCKING` with 10 findings: 4 shared with the Claude fan, 6 `unique-real`, none `false`.
It found 4 of the fan's 12 findings and 1 of its 2 `BLOCKING` ones (card 23 shortening `PATTERN.md` after the cards that grow it); it missed the hand-enumerated readers of cards 1, 6 and 14.
The report sets the row beside the earlier `P1` rows and this setup's `P4` row.
The rollouts show three concurrent forks that inherited the handler's context, and an `AGENTS.md` load in every rollout.
The run took 1114 s and 412k fresh input tokens across the handler and its forks; the weekly window read 18 percent at the first request and 28 at the end, with two sibling benches drawing on the same account.

## Design decisions

- The harness is copied from the `P4` bench with only the repository root and harness directory changed, and `tuned-block.md` is byte-identical, so the two benches compare.
- The lenses are the ones the fan's round 1 on `P1` recorded: `affected-set`, `guard-reach` and `way-forward`.
- Findings are graded as `shared`, `unique-real` or `false` against the fan's `P1` findings, with each unmatched claim checked on the snapshot's cards and code in `data/match.tsv`.
- One input, one run and one grader limit what the numbers show; the report lists those limits.
