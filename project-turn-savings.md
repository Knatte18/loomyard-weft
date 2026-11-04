# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-04 15:14 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 2d · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Measure how Plan and Discussion writers explore and whether they use lookup subagents**<br>• From 10 landed tapestry runs, measure the Plan and Discussion writers exploration turns and their use of lookup subagents | [`planner-fork-lookups`](design-planner-fork-lookups.md) | darn · docs | Drift-Verify | neve | 2d<br>10m | loom |
| T2 | **The batching hook counts Bash lookups and reaches forks**<br>• Lookup chain counts read-only Bash commands; the hook judges subagent and fork calls against their own transcript | [`batching-hook-reach`](design-batching-hook-reach.md) | darn · light | Finalize | neve | 41m<br>41m | claude, shuttle |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Re-measure the batching hook and the edit-round rule a week after the hook reaches forks**<br>• run-all.sh on a fresh sample: firing rate, batching after a firing, edit chains split by gate fix against mid-round | [`batching-hook-remeasure`](design-batching-hook-remeasure.md) | 41m<br>41m | tools, claude |
