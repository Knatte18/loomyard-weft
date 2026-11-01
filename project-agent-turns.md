# P2 Fewer agent turns, Webster forks first

Rendered 2026-11-01 16:27 UTC; every age below is as of then.
Under: [Board](README.md).

[`agent-turns`](design-agent-turns.md) · Age 2h · Owner `neve`

Cut the turns agents spend on shell edits, piecemeal gate loops and sliced reads, one measured fix at a time

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T9 | **Agents read a file whole or all its spans in one message, not in slices**<br>• No reading one file in three to five slices over as many turns, nor re-reading it | [`read-whole-not-slices`](design-read-whole-not-slices.md) | darn | Finalize | neve | 2h<br>1h | webster, stencil |
| T13 | **Trial: the batching instruction in the rendered CLAUDE.local.md through heddle**<br>• Put the batching instruction into heddle's claude-file target and measure calls per turn with forkaudit against today's 1.7 | [`batching-in-claude-local`](design-batching-in-claude-local.md) | darn | Finalize | neve | 2h<br>1h | claude, heddle |
| T16 | **The round-1 Plan reviewer gathers in one call a turn**<br>• 1.05 calls a turn and 41% of its weight removable by merging, though it carries the batching sentence | [`plan-reviewer-batching`](design-plan-reviewer-batching.md) | darn | PR-Review | neve | 1h<br>1h | burler, stencil |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T14 | **Trial: agents settle every edit before the first, then gate once**<br>• An imperative heddle instruction: read, decide all edits, make them, gate once and fix the whole report in one round | [`plan-edits-before-gate`](design-plan-edits-before-gate.md) | darn | T13 | 1h<br>3m | heddle, stencil, claude |

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F12 |  | **A card gate failure goes to a cold, cheap fixer instead of the fork**<br>• Go spawns a fresh fixer with the gate report, the card and its diff, so the fork's large context does not run the fix loop | [`card-gate-cold-fixer`](design-card-gate-cold-fixer.md) | 2h<br>2h | webster, gate |
