# Support log — board-entry-handles

## Interview

A darn run holds no interview: the board entry's operator decision is the whole statement of the change, and the writer made the remaining calls itself.

## Rejected alternatives

- A `board.yaml` key mapping each type to its letter: more config surface and a hub file edit at landing, while the first letter of each loomyard type already gives the operator's letters.
- Deriving the handle on read from the entry's position: the operator decision requires a stored handle that adding or removing other entries never changes.
- Clearing task priority with a one-off migration write: a run-held task carrying a priority would make that write refuse under the run lock; dropping it on load avoids the lock.
- Accepting handles in `depends_on`, `remove_slugs` and intake's `into`: these are stored slug lists or slug fields, and resolving handles inside them would store or compare a value that changes on promote.
- A two-letter prefix on a first-letter collision: no hub needs it, and it complicates the handle shape the bare argument parses.

## Review rounds

_No rounds yet._

## Question ledger

- Where does a type's letter come from? Self-picked: the upper-cased first letter of the type label, collisions refused at config load.
- Which verbs take a handle? Self-picked: every slug|id selector, plus a bare handle argument.
- How are existing task priorities removed? Self-picked: dropped on load, persisted by the next write.
- How do Done entries order? Self-picked: tasks first, then notes, each by handle.
- What does the README priority column show? Self-picked: `H`, `L` or empty under a `P` header, notes only.
