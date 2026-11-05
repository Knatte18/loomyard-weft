# Re-measure the batching hook and the edit-round rule a week after the hook reaches forks

`batching-hook-remeasure` · Task · research, tools, claude · running · Seed-Child

## Idea (operator and hub orch, 2026-11-04)

Re-run the turn-savings measurement once about a week of runs has landed with task `batching-hook-reach`, using `docs/research/turns/turn-savings-measure-2026-11-04/run-all.sh` on a fresh sample, no new agent run.

- Count hook firings, by breach and by main session against fork, and whether the next message batched; with a few dozen firings this can be read as a rate.
- Split the edit-chain turns (edit-only turns following an edit-only turn above 150k) by what they hold: edits after a failed gate, against edits in the middle of a round of edits. Edit follows rose from 2.5% to 4.2% of context in the 2026-11-04 sample, against the heddle entry `lym-plan-edits-before-gate`'s "make all of those edits in one message" rule; the split shows whether that rule works.
- If the hook still shows little effect, the report recommends removing it.
- Cap: at most 20 darn runs and 5 tapestry runs, the most recent landed after `batching-hook-reach`.
- Runs as darn with profile `docs`.
