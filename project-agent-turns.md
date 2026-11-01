# P2 Fewer agent turns, Webster forks first

Rendered 2026-11-01 14:46 UTC; every age below is as of then.
Under: [Board](README.md).

`agent-turns` · Age 29m · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Agents change files with the Edit tool, never shell edits**<br>• Read a file to change with Read and edit it with Edit, all its edits in one message; a guard refuses sed -i and its kin | [`edit-with-edit-tool`](design-edit-with-edit-tool.md) | darn | Code-Burler-Fix | neve | 29m<br>19m | process, stencil, claude |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **The gate reports every failure at once, lint included, in one readable result**<br>• lym gate test prints a compact list of every build, test and lint failure with file:line and values, and the log path | [`gate-failure-report`](design-gate-failure-report.md) | darn | T6 | 29m<br>19m | gate, lint |

## Notes (6)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (5)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E13 |  | **Trial: agents settle every edit before the first, then gate once**<br>• An imperative heddle instruction: read, decide all edits, make them, gate once and fix the whole report in one round | [`plan-edits-before-gate`](design-plan-edits-before-gate.md) | 0m<br>0m | heddle, stencil, claude |
| E15 |  | **A card that retires a name sweeps the whole tree, tests included, first**<br>• The implementer greps every retired name across the tree and test data before its first edit, and reads and edits all hits in one message each | [`retired-name-sweep`](design-retired-name-sweep.md) | 29m<br>29m | webster, stencil |
| E17 |  | **Agents read a file whole or all its spans in one message, not in slices**<br>• No reading one file in three to five slices over as many turns, nor re-reading it | [`read-whole-not-slices`](design-read-whole-not-slices.md) | 29m<br>29m | webster, stencil |
| E18 |  | **Agents still try the forms lym's guards refuse, at a turn each**<br>• Python and raw go build or go vet attempts are refused and retried; find why the steering does not hold and close it | [`hook-refused-turns`](design-hook-refused-turns.md) | 28m<br>24m | process, claude |
| E21 |  | **Trial: the batching instruction in the rendered CLAUDE.local.md through heddle**<br>• Put the batching instruction into heddle's claude-file target and measure calls per turn with forkaudit against today's 1.7 | [`batching-in-claude-local`](design-batching-in-claude-local.md) | 28m<br>28m | claude, heddle |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F12 |  | **A card gate failure goes to a cold, cheap fixer instead of the fork**<br>• Go spawns a fresh fixer with the gate report, the card and its diff, so the fork's large context does not run the fix loop | [`card-gate-cold-fixer`](design-card-gate-cold-fixer.md) | 29m<br>28m | webster, gate |

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T5 | **Study: turns and context of the Discussion and Plan review sessions**<br>• Measure where reviewer, fixer and judge turns and context go on three tapestry runs, and what Sonnet fixers would cost<br>• `profile: docs` | [`review-turn-audit`](design-review-turn-audit.md) | 28m<br>19m | burler |
