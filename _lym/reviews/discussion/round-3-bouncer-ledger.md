---
round: 3
ledger:
  - key: d2-pause-does-not-hold-inner-run
    rounds: [1]
    status: resolved
  - key: cancel-archive-tags-not-on-origin
    rounds: [1, 2]
    status: resolved
  - key: no-check-new-parent-takes-member
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-ref-workaround-outside-procedure
    rounds: [1]
    status: resolved
  - key: partial-finish-vs-refusal-undefined
    rounds: [1, 2]
    status: resolved
  - key: h2-move-note-in-stored-record
    rounds: [1]
    status: resolved
  - key: glossary-move-note-names-writer
    rounds: [1]
    status: resolved
  - key: move-note-file-barred-for-writer
    rounds: [2]
    status: resolved
---

Round 3 resolved the archive-tag wait (F10), the partial-finish classification (F9) and the writer's note file (F8).
One key remains: the parent check for a flagged beam.
Its round 1 form was a board-side check, its round 2 form the beam ready flag in the beam worktree (F11), and its round 3 form the remote read of that record (F12).
F12 says the record is committed on the beam's records branch `<beam branch>-weft` and never pushed, so the read on the code branch finds nothing.
