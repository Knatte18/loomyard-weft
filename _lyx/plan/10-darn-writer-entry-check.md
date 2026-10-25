# Card 10 — darn-writer-entry-check

**Prosa:**
- `contracts/stencils/darn/darn-template-write.md`

**Intent:** D10: the darn writer checks its entry against the board, the in-flight runs and the parent's landed commits before it writes its decision record, and asks its parent when the entry is plainly outdated.

Add to Step 3, before the paragraph that writes the record and under Step 3's own condition (the record or support log absent, or `lyx loom validate-discussion` reporting findings):

- Read `lyx board list`, every entry that touches the task and every board note titled `Decision for <slug>:` for it, and `lyx loom peers` for the runs in flight and the parent's commits since the fork.
- When the entry is plainly outdated, because landed work overtook it or another board entry or run contradicts it, and no decision note answers it, send your parent a message by `SendMessage` naming what overtook or contradicts the entry, and ask it to write its answer as a `Decision for <slug>:` board note and then reply by `SendMessage`, since a note alone wakes nothing; then end your turn to wait.
  On any message from your parent, re-read the board for the note.
- Write the answer into the decision record as a Decision citing the parent.
- Once the record is committed, ask your parent to record the same answer with `lyx loom decision add`; until the record carries that `Added after Discussion` entry, name the answer in the change description's design decisions as an unrecorded parent correction, as the scope-correction section does.
- With no parent, do not wait and change nothing: write the change description, your one output file, as a stop report naming the outdated entry, what overtook or contradicts it, and the way forward, a `Decision for <slug>:` board note that answers it and then `lyx loom resume`; then end your turn.
  When the handoff's `discussion` gate re-prompts you while that stop still holds, end your turn with the stop report unchanged, so the gate exhausts and halts the run with its findings kept.
- An ordinary misalignment you settle yourself, as today.

Bound, stated nowhere else in the prompt: the parent's reply wakes the writer, and otherwise `writer_timeout_min` ends the session; nothing respawns it.
Static prose: no test reads it, and no marker changes.
