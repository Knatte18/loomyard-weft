# Board entries get short handles (T4, B2, E7) for lookup; tasks lose priority; tighter README tables

`board-entry-handles` · Task · enhancement, board

Depends on: [`machine-handover`](design-machine-handover.md)

## Seen (operator, 2026-10-22)

The operator reads the board README and wants to name an entry to the orch without typing its long slug.
README rows are numbered 1, 2, 3 per table, so a row number shifts whenever an entry is added or removed and names nothing stable.
Every entry already has a stored `id`, but it is shared across all kinds and types and runs into the hundreds.
A task's priority only distracts: a note that became a task is prioritized by being one.
The README note tables are short of width.

## Operator decision (2026-10-22)

1. Tasks carry no priority; only notes do. `lyx board promote` clears it, and an upsert or merge that gives a task a priority is refused.
2. Every entry carries a short handle stored in the board database, one counter per kind or type: `T` for a task, and for a note its type label, `B` for `bug`, `E` for `enhancement`, `F` for `feature`, `A` for `audit`.
   - A new entry gets the lowest number of its letter not in use, so numbers stay small; a number freed by a removed entry is given out again.
   - A handle stays fixed while its entry lives: adding or removing other entries never changes it.
   - No history: a note promoted to a task drops its note handle and gets the lowest free `T` number.
   - Existing entries get handles once, when this lands, in their creation order.
3. Every board verb that takes a slug or id also takes a handle, such as `lyx board get B4`.
4. README:
   - The handle replaces the row number.
   - Notes stand in one table per type, sorted by handle, with a narrow priority column: `H` for high, `L` for low, empty for normal.
   - Tasks keep today's groups (running, ready, dependency layers), each sorted by handle.
   - Age and revised share one column, age above revised in the same cell (`<br>`), its header on two lines the same way.
5. `lyx board list --text` prints the handle first.

## Orch recommendation, not a decision

Type labels are hub config (`board.yaml` `types`), so the letter of each type should come from the hub's config, with the letters above as the loomyard hub's values, rather than a list hard-coded in lyx.
