# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-02 13:08 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 18h · Owner `osl-1046`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **One cold, cheap fixer clears a failed writer gate instead of the writer at full context**<br>• When the Darn-Write, plan, discussion or Webster card gate fails, Go spawns one shared cold fixer with the gate report, generalised from webster fix-verify | [`cold-gate-fixer`](design-cold-gate-fixer.md) | tapestry · light | Plan-Burler-Fix | osl-1046 | 18h<br>18h | webster, loom, darn, gate |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 22h<br>18h | tools |
| R2 |  | **Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant**<br>• Bench the guidance that sends Plan and Discussion writers' lookups to parallel Sonnet subagents against a writer given none | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 18h<br>4m | loom |
| R3 |  | **Measure the effect of the batching breach hook**<br>• After the batching breach hook lands, compare breach turns and context tokens against the 28-session baseline and tune its thresholds | [`batching-hook-effect`](design-batching-hook-effect.md) | 1h<br>4m | claude, shuttle, tools |
