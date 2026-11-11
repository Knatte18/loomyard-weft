# Decision records should allow editing a decision, not only appending

`decision-record-edit` · Note · enhancement, loom, board

Imported from [issue #117](https://github.com/Knatte18/loomyard/issues/117).

Seen: 'lym loom decision add' only appends. When a later call changes an earlier decision, the record keeps both: the original D-entry and an 'Added after Discussion' entry that overrides it. In a design that went through several review rounds and operator calls (motif run, then two independent reviews), the record grew about 30 appended entries, several contradicting the D-entries above them, so the record stopped being a readable statement of the design. The operator keeps board bodies as the current design only, and wants records kept the same way.

Expected: a verb to edit or replace a recorded decision in place (for example 'lym loom decision edit <slug> D7 --from <file>', and 'decision remove'), committed and announced like 'add', with git history keeping the old text. The same for a Sketch's stored record after its motif run, so a promoted Sketch's raddle reads the current design.

Related: #114 (no verb replaces a Sketch's record), #116 (raddle validates against the stale record and refuses a split that follows the body).