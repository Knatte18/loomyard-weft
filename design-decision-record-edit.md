# Decision records should allow editing a decision, not only appending

`decision-record-edit` · Note · enhancement, loom, board

Imported from [issue #117](https://github.com/Knatte18/loomyard/issues/117).

Seen: 'lym loom decision add' only appends. When a later call changes an earlier decision, the record keeps both: the original D-entry and an 'Added after Discussion' entry that overrides it. In a design that went through several review rounds and operator calls (motif run, then two independent reviews), the record grew about 30 appended entries, several contradicting the D-entries above them, so the record stopped being a readable statement of the design. The operator keeps board bodies as the current design only, and wants records kept the same way.

Expected: a verb to edit or replace a recorded decision in place (for example 'lym loom decision edit <slug> D7 --from <file>', and 'decision remove'), committed and announced like 'add', with git history keeping the old text. The same for a Sketch's stored record after its motif run, so a promoted Sketch's raddle reads the current design.

Related: #114 (no verb replaces a Sketch's record), #116 (raddle validates against the stale record and refuses a split that follows the body).

## From issue #114

Imported from [issue #114](https://github.com/Knatte18/loomyard/issues/114).

## Seen

`lym loom decision add` appends each decision under an "Added after Discussion" heading at the end of the record's Decisions section.
The Discussion fixers then edit the main decisions to match, but the appended entries stay.
A record that took many parent and arbiter decisions ends with dozens of appended entries after the main decisions, some overriding earlier appended ones (for example a corrected reference commit after the wrong one).
The stored record of the resulting Sketch therefore carries superseded design next to the current one, and a reader cannot tell which binds without reading every entry in order.

Once the motif run is done, there is also no verb to replace a Sketch's stored decision record.
`lym board get --record` prints it, but nothing writes it, so further review findings on a Sketch cannot be folded into the record before the Sketch is promoted to raddle.

Workaround used: the full current design, rewritten as one document with every decision and later finding folded in, is written into the Sketch's body with `lym board upsert`; the stale stored record remains beside it.

## Expected

- A decision added after Discussion is folded into the record's main decisions by the fixer, and its appended entry is removed once folded, so the record holds the current design only.
- A verb to replace a Sketch's (or any discussed entry's) stored decision record from a file, such as `lym board upsert ... --record-file <file>`, refused while a run holds the entry.
