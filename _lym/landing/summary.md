# Report the first Codex assistant review run with its lens fan

Adds a research report and its data under `docs/research/codex/review/assist-fan-trial-2026-11-06/`; no source or contract changes.
The assistant reviewer had run solo on Codex, but the fan (`code_assist_reviewer_fan`) never had, and no lym fork audit covers the subagents the Codex `assist-fan` stencil has Codex spawn with `spawn_agent`.
This trial answers whether Astra with lenses works.

One `lym shuttle run --model "astra[medium]"` reviewed the landed commit `b93a0a52` with the three lenses of the `review-code` fan.
The report records:

- The fan worked: one subagent started per lens, all three returned, and the assistant waited for them with `wait_agent` inside its one turn.
- The consolidated review passes `lym burler validate-review`.
- The turn took about 266 s; token figures for the handler and each lens come from Codex's own session records.
- It returned three `MEDIUM` findings, each checked against the tree and found real.
  The report compares them with the earlier review of the same change, and says the comparison is rough because that review read the code before its fixes.

`data/` holds the prompt, the review and the three lens reports verbatim.

Closes: none; the task has no linked issue.

## Design decisions

- The prompt was composed by hand from the stencils an assistant reads, since no verb composes it outside a round.
  The rubric is a four-line stand-in.
  The report names this as a limit.
- Two wording points in the Codex `assist-fan` block are filed as notes, not fixed: the block says `wait` where Codex's tool is `wait_agent`, and says to spawn "all in one go" where Astra issued the spawns as separate calls over 29 s.
  Neither broke the run.

## To check by hand

- Part 2 of the task, the real composed seat, is not in this change: this task's `run_settings` run Code-Review with the assistant fanned, and the report names that part as still to come.
- The report could not check that the subagent prompts held only the boilerplate and the lens text, since they are encrypted in the session record.
- One run on one commit: the report makes no claim that the fan finds this much on a typical diff or twice in a row.
