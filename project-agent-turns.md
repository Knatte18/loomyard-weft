# P2 Fewer agent turns, Webster forks first

Rendered 2026-11-01 14:16 UTC; every age below is as of then.
Under: [Board](README.md).

`agent-turns` · Age 0m · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (5)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (4)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E13 |  | **Agents change files with the Edit tool, never shell edits**<br>• Read a file to change with Read and edit it with Edit, all its edits in one message; a guard refuses sed -i and its kin | [`edit-with-edit-tool`](design-edit-with-edit-tool.md) | 0m<br>0m | process, stencil, claude |
| E14 |  | **The gate reports every failure at once, lint included, in one readable result**<br>• lym gate test prints a compact list of every build, test and lint failure with file:line and values, and the log path | [`gate-failure-report`](design-gate-failure-report.md) | 0m<br>0m | gate, lint |
| E15 |  | **A card that retires a name sweeps the whole tree, tests included, first**<br>• The implementer greps every retired name across the tree and test data before its first edit, and reads and edits all hits in one message each | [`retired-name-sweep`](design-retired-name-sweep.md) | 0m<br>0m | webster, stencil |
| E17 |  | **Agents read a file whole or all its spans in one message, not in slices**<br>• No reading one file in three to five slices over as many turns, nor re-reading it | [`read-whole-not-slices`](design-read-whole-not-slices.md) | 0m<br>0m | webster, stencil |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F12 |  | **A card gate failure goes to a cold, cheap fixer instead of the fork**<br>• Go spawns a fresh fixer with the gate report, the card and its diff, so the fork's large context does not run the fix loop | [`card-gate-cold-fixer`](design-card-gate-cold-fixer.md) | 0m<br>0m | webster, gate |
