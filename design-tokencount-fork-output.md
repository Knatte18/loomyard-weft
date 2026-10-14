# tokencount under-counts fork output tokens

`tokencount-fork-output` · Task · bug · running · Finalize

## Seen (`twill-fork-timing` bench, 2026-10-13)

tokencount under-counts subagent (fork) output tokens.
It keeps the first transcript line of each message id, and in a subagent transcript that first line carries a partial `output_tokens` (6 on a thinking block against 4,546 on the message's final line).
Writer (main session) figures match exactly; fork figures come out low: one bench run's forks $1.49 by tokencount against $2.09 counted from each message's last line, another $1.15 against $1.62.
Detail in the bench report's Deviations (`docs/research/twill-fork-timing-bench.md`).

## Affected

Every cost figure for forks taken with tokencount is low: the twill benches' fork costs, `docs/research/token-usage-by-role.md` and the estimates in `model-downgrade` that rest on it.
