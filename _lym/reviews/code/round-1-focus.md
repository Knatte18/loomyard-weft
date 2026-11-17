---
round: 1
focus:
  - "internal/shuttleengine/editloop.go fold: settle whether the repeat count matches the board entry's idea, 'content returning to an earlier hash'. A file's pre-edit content is never hashed, so check an A to B to A back-and-forth, and a same-`old_string` repeat that changes the content each time."
  - "internal/shuttleengine/editloop.go fold: settle the window pruning and the reset after a notice. Check that signals with no `At`, an empty `ContentHash`, or an interleaved agent id and file path cannot miscount, and that `histories` is cloned and written back correctly."
  - "internal/shuttleengine/claudeengine/editrecord.go RecordEdit and fileContentHash: settle whether hashing at hook time is sound when the hook runs after a later edit, with a relative file path, a large file, or an unreadable file. Check the hash lands in the record and reaches `SessionSignal.ContentHash` through signals.go."
  - "internal/shuttleengine/reinterpret.go DropEditLoopEdits and its registration in internal/configreg/configreg.go: settle whether a hub's old `edit_loop_edits` is dropped once and not on later runs, and that a scalar, null or missing key and a malformed document are handled. Check the hub default of 0 the board entry names is replaced by the template's working threshold."
  - "Scope against the board entry: settle whether the diff changes anything beyond the entry's 'Idea' and the threshold reset, and whether docs/overview.md and the package doc.go files carry the new counting semantics, including the `edit_loop_edits` template text and the notice wording."
  - "Test economy across editloop_test.go, editrecord_test.go, reinterpret_test.go, configreg_test.go and config_test.go: settle whether each behavior the diff adds is covered, in particular the progress-making fixer case from the board entry (several edits, new content each time, no notice) and the revert case."
---

Seed focus for round 1.
The decision record holds only the board entry under `## Goal`; the other sections are empty, so the entry is the measuring stick.
