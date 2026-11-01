# P1 Fewer agent turns, fewer tokens

Rendered 2026-11-01 18:37 UTC; every age below is as of then.
Under: [Board](README.md).

`turn-savings` · Age 20m · Owner `neve`

Token savings by cutting agent turns, above all late turns at large context

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **Agents may use subagents and forks; only headless claude -p stays refused**<br>• Remove the Agent tool deny, refuse headless claude -p launches instead, and tell writers when to hand work to Sonnet subagents | [`darn-sonnet-helpers`](design-darn-sonnet-helpers.md) | darn | Darn-Write | neve | 58m<br>6m | claude, shuttle, stencil |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (1)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | `draft` **Shorten the plan spec and split it into sub-files read by need**<br>• loom-plan-spec.md is 112 KB, half of it a catalogue of validation checks; cut it down and split it so each role reads only what it uses | [`plan-spec-split`](design-plan-spec-split.md) | darn |  | 31m<br>0m | loom, stencil, webster |

## Notes (4)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Measure the effect of the agent-turns project**<br>• Turns, calls per turn and cost before and after P2 agent-turns, for darn writers, Webster forks and review roles | [`agent-turns-effect`](design-agent-turns-effect.md) | 4h<br>20m | tools |
| R2 |  | **Plan writer hands its code lookups to fresh subagents and keeps the overview**<br>• The Plan writer pays its whole context for each late grep; research whether forks answering batched questions cut that cost without losing the plan overview | [`planner-fork-lookups`](design-planner-fork-lookups.md) | 15m<br>6m | loom |

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E7 |  | **A hook nudges an agent back to batching the moment it breaks the rule**<br>• Above about 150k context agents drift into grep-then-slice chains and one-fix-per-turn edits, about a third of context tokens in a 28-session sample; a PostToolUse hook adds one line on a breach, with sharper rule wording | [`batching-breach-nudge`](design-batching-breach-nudge.md) | 28m<br>20m | claude, shuttle |
| E8 |  | **A cold, cheap fixer clears a failed writer gate instead of the writer at full context**<br>• When the Webster card gate, the plan gate or the discussion gate fails, Go spawns a cold fixer with the gate report rather than the writer fixing at its large context | [`cold-gate-fixer`](design-cold-gate-fixer.md) | 15m<br>15m | webster, loom, gate |
