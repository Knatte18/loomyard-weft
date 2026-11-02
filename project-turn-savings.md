# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-02 12:12 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 17h · Owner `osl-1046`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **A hook nudges an agent back to batching the moment it breaks the rule**<br>• Above about 150k context agents drift into grep-then-slice chains and one-fix-per-turn edits, about a third of context tokens in a 28-session sample; a PostToolUse hook adds one line on a breach, with sharper rule wording | [`batching-breach-nudge`](design-batching-breach-nudge.md) | darn | Code-Burler-Fix | osl-1046 | 18h<br>44m | claude, shuttle |
| T5 | **One cold, cheap fixer clears a failed writer gate instead of the writer at full context**<br>• When the Darn-Write, plan, discussion or Webster card gate fails, Go spawns one shared cold fixer with the gate report, generalised from webster fix-verify | [`cold-gate-fixer`](design-cold-gate-fixer.md) | tapestry · light | Drift-Baseline | osl-1046 | 17h<br>17h | webster, loom, darn, gate |
| T6 | **Discussion writers hand lookups to Sonnet subagents, and the subagent model is config**<br>• Add the Plan stencil's explore-subagent guidance to the Discussion-Write stencil, and read the subagent model from a loom config key instead of a literal in each stencil | [`explore-subagent-model`](design-explore-subagent-model.md) | darn · light | Darn-Write | osl-1046 | 31m<br>31m | stencil, loom, darn, config |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 21h<br>17h | tools |
| R2 |  | **Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant**<br>• Bench the guidance that sends Plan and Discussion writers' lookups to parallel Sonnet subagents against a writer given none | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 17h<br>31m | loom |
| R3 |  | **Measure the effect of the batching breach hook**<br>• After the batching breach hook lands, compare breach turns and context tokens against the 28-session baseline and tune its thresholds | [`batching-hook-effect`](design-batching-hook-effect.md) | 44m<br>44m | claude, shuttle, tools |
