# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-02 11:19 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 17h · Owner `osl-1046`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **One cold, cheap fixer clears a failed writer gate instead of the writer at full context**<br>• When the Darn-Write, plan, discussion or Webster card gate fails, Go spawns one shared cold fixer with the gate report, generalised from webster fix-verify | [`cold-gate-fixer`](design-cold-gate-fixer.md) | tapestry · light |  | 16h<br>16h | webster, loom, darn, gate |

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 21h<br>16h | tools |
| R2 |  | **Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant**<br>• Plan-Write and Discussion-Write start parallel cold-start explore agents that return a list of relevant files and facts; the writer reads only those and writes; encouraged, not commanded; bench guidance against none | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 16h<br>15h | loom |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E7 |  | **A hook nudges an agent back to batching the moment it breaks the rule**<br>• Above about 150k context agents drift into grep-then-slice chains and one-fix-per-turn edits, about a third of context tokens in a 28-session sample; a PostToolUse hook adds one line on a breach, with sharper rule wording | [`batching-breach-nudge`](design-batching-breach-nudge.md) | 17h<br>17h | claude, shuttle |
