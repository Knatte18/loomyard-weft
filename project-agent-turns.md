# P2 Fewer agent turns, Webster forks first

Rendered 2026-11-01 15:10 UTC; every age below is as of then.
Under: [Board](README.md).

`agent-turns` · Age 54m · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (8)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **The gate reports every failure at once, lint included, in one readable result**<br>• lym gate test prints a compact list of every build, test and lint failure with file:line and values, and the log path | [`gate-failure-report`](design-gate-failure-report.md) | darn | T6 | 54m<br>43m | gate, lint |

### Draft (7)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **A card that retires a name sweeps the whole tree, tests included, first**<br>• The implementer greps every retired name across the tree and test data before its first edit, and reads and edits all hits in one message each | [`retired-name-sweep`](design-retired-name-sweep.md) | darn |  | 54m<br>0m | webster, stencil |
| T9 | `draft` **Agents read a file whole or all its spans in one message, not in slices**<br>• No reading one file in three to five slices over as many turns, nor re-reading it | [`read-whole-not-slices`](design-read-whole-not-slices.md) | darn | T8 | 54m<br>0m | webster, stencil |
| T12 | `draft` **Agents still try the forms lym's guards refuse, at a turn each**<br>• Python and raw go build or go vet attempts are refused and retried; find why the steering does not hold and close it | [`hook-refused-turns`](design-hook-refused-turns.md) | darn |  | 52m<br>0m | process, claude |
| T13 | `draft` **Trial: the batching instruction in the rendered CLAUDE.local.md through heddle**<br>• Put the batching instruction into heddle's claude-file target and measure calls per turn with forkaudit against today's 1.7 | [`batching-in-claude-local`](design-batching-in-claude-local.md) | darn |  | 52m<br>0m | claude, heddle |
| T14 | `draft` **Trial: agents settle every edit before the first, then gate once**<br>• An imperative heddle instruction: read, decide all edits, make them, gate once and fix the whole report in one round | [`plan-edits-before-gate`](design-plan-edits-before-gate.md) | darn |  | 24m<br>0m | heddle, stencil, claude |
| T15 | `draft` **A reviewer or fixer wakes once for all its forks, not once per fork**<br>• 79 no-call wake-up turns at full context over three runs, 8-9% of the forking review roles weight | [`fork-results-one-wake`](design-fork-results-one-wake.md) | darn |  | 18m<br>0m | burler, stencil |
| T16 | `draft` **The round-1 Plan reviewer gathers in one call a turn**<br>• 1.05 calls a turn and 41% of its weight removable by merging, though it carries the batching sentence | [`plan-reviewer-batching`](design-plan-reviewer-batching.md) | darn |  | 18m<br>0m | burler, stencil |

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F12 |  | **A card gate failure goes to a cold, cheap fixer instead of the fork**<br>• Go spawns a fresh fixer with the gate report, the card and its diff, so the fork's large context does not run the fix loop | [`card-gate-cold-fixer`](design-card-gate-cold-fixer.md) | 54m<br>52m | webster, gate |
