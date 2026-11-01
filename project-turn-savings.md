# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-01 19:12 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 54m · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Agents may use subagents and forks; only headless claude -p stays refused**<br>• Remove the Agent tool deny, refuse headless claude -p launches instead, and tell writers when to hand work to Sonnet subagents | [`darn-sonnet-helpers`](design-darn-sonnet-helpers.md) | darn | Finalize | neve | 1h<br>41m | claude, shuttle, stencil |
| T4 | **Shorten the plan spec and split it into sub-files read by need**<br>• loom-plan-spec.md is 112 KB, half of it a catalogue of validation checks; cut it down and split it so each role reads only what it uses | [`plan-spec-split`](design-plan-spec-split.md) | darn | Finalize | neve | 1h<br>34m | loom, stencil, webster |
| T11 | **tokencount counts darn runs and compares recipes**<br>• Count darns with their own segments, so a darn's cost stands beside a tapestry's | [`tokencount-darn-runs`](design-tokencount-darn-runs.md) | darn | Darn-Write | neve | 4h<br>19m | tools |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | `draft` **One cold, cheap fixer clears a failed writer gate instead of the writer at full context**<br>• When the Darn-Write, plan, discussion or Webster card gate fails, Go spawns one shared cold fixer with the gate report, generalised from webster fix-verify<br>• `profile: light` | [`cold-gate-fixer`](design-cold-gate-fixer.md) | tapestry | T4 | 49m<br>26m | webster, loom, darn, gate |

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 4h<br>26m | tools |
| R2 |  | **Plan and Discussion writers explore through parallel Sonnet agents that return only what is relevant**<br>• Plan-Write and Discussion-Write start parallel cold-start explore agents that return a list of relevant files and facts; the writer reads only those and writes; encouraged, not commanded; bench guidance against none | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 49m<br>7m | loom |

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E7 |  | **A hook nudges an agent back to batching the moment it breaks the rule**<br>• Above about 150k context agents drift into grep-then-slice chains and one-fix-per-turn edits, about a third of context tokens in a 28-session sample; a PostToolUse hook adds one line on a breach, with sharper rule wording | [`batching-breach-nudge`](design-batching-breach-nudge.md) | 1h<br>54m | claude, shuttle |
