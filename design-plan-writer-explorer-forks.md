# The Plan writer explorer forks cost more than the writer and show no gain

`plan-writer-explorer-forks` · Note · enhancement, loom, stencil

## What was seen

`tools/tokencount -since 2026-11-03` over the 21 tapestry runs since then (2026-11-18):
the Plan writer's forks cost $137.84, more than the Plan writer itself ($126.24), about $6.50 per run.
tokencount counts every subagent transcript under the Plan writer as a fork, so this figure mixes the twill-plan forks (one fork writes the plan, one fork per lens reviews it) with the exploration subagents the writer spawns through the Agent tool. The split is not measured yet; measure it first.
The operator judges they add little to the plan beyond their cost.

## Idea

Stop the Plan writer from spawning exploration subagents through the Agent tool; it reads the code itself. Twill stays: it makes the first plan version much better than a solo writer (operator, 2026-11-18; `docs/research/loom/plan-writer-bench-2026-11-12/report.md`).
The plan review stays as it is.

## Check afterwards

Compare the next tapestry runs' plan review rounds and blocking findings with the runs counted above.
If they rise, turn the explorer subagents back on.
