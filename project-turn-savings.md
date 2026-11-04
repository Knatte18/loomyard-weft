# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-04 14:20 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 2d · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T9 | **Measure agent-turns and the batching hook from landed runs**<br>• One report: turns, calls per turn, gate loops, refusals and batching-hook breaches over up to 20 darn and 5 tapestry runs landed since 2026-11-02 | [`turn-savings-measure`](design-turn-savings-measure.md) | darn · docs | Finalize | neve | 3d<br>22m | tools, claude, shuttle |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R2 |  | **Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant**<br>• Bench the guidance that sends Plan and Discussion writers' lookups to parallel Sonnet subagents against a writer given none | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 2d<br>2d | loom |
