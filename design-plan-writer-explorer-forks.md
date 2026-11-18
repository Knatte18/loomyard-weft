# The Plan writer explorer forks cost more than the writer and show no gain

`plan-writer-explorer-forks` · Note · enhancement, loom, stencil

## What was seen

`tools/tokencount -since 2026-11-03` over the 21 tapestry runs since then (2026-11-18):
the Plan writer's forks cost $137.84, more than the Plan writer itself ($126.24), about $6.50 per run.
They are the exploration subagents the Plan writer was set up to spawn.
The operator judges they add little to the plan beyond their cost.

## Idea

Stop the Plan writer from spawning exploration subagents; it reads the code itself and writes one version of the plan.
The plan review stays as it is and is the safety net: a plan that misses something costs at most an extra review round.

## Check afterwards

Compare the next tapestry runs' plan review rounds and blocking findings with the runs counted above.
If they rise, turn the explorer subagents back on.
