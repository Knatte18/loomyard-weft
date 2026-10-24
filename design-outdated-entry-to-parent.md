# A row before Discussion aligns the task text against the board, peers and landed work, and adjusts it

`outdated-entry-to-parent` · Note · enhancement, loom, batten, board

## Seen

A task's board entry is written when it is promoted, and can sit for days while other work lands and other runs start.
Its text reaches Discussion-Write as it stands, so a scope that overlaps, contradicts or has been overtaken by other planned or landed work is found only in the Discussion perch or by the alignment agent, after a whole Discussion has been written; a scope change costs more the later it comes.

## Idea

A row of the run, before Discussion-Write (and before Darn-Write), aligns the task's own entry text against the rest of the board, the other in-flight runs' records (`lyx loom peers`) and the parent's recent landed commits, and adjusts the text before anything is written from it.

- The row settles misalignments itself and writes the adjusted text into the entry: a reference to work that has landed, a dependency left unnamed, an overlap with other planned work trimmed or made explicit.
- It escalates to the orch only for something larger, such as a design conflict with another run or a task that no longer makes sense; the orch is not to be disturbed by ordinary misalignments (operator, 2026-10-22).

The board's run lock exists to stop the orch from editing a started run's entry, which is pointless once the run has read it (operator, 2026-10-22); it does not need to stop the run's own pre-Discussion row, since the text still matters then.
So this row edits its own entry through an exemption the lock grants it.
The rule in this repo's `CLAUDE.md` that no decision, card or step of a run changes the running task's own entry goes: a rule about lyx's own behaviour cannot live in one repo's `CLAUDE.md`, since it does not travel with lyx to other hubs (operator, 2026-10-22); the lock and its exemption are the mechanism, documented in their package's `doc.go`.

Related: note `discussion-alignment-into-lenses`, which drops the alignment agent at the discussion point; with this row in place the perch's `board-fit` only checks what the discussion itself added.
Operator idea, 2026-10-22; the design is open.
