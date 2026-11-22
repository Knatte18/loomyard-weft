---
round: 2
ledger:
  - key: d2-pause-does-not-hold-inner-run
    rounds: [1]
    status: resolved
  - key: cancel-archive-tags-not-on-origin
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-check-new-parent-takes-member
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-ref-workaround-outside-procedure
    rounds: [1]
    status: resolved
  - key: partial-finish-vs-refusal-undefined
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: h2-move-note-in-stored-record
    rounds: [1]
    status: resolved
  - key: glossary-move-note-names-writer
    rounds: [1]
    status: resolved
  - key: move-note-file-barred-for-writer
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised seven findings; round 2 resolved four of them.
The hold is now `lym tender pause <slug>` and the stored-record carrier and glossary line are settled.
Three round 1 keys persist in narrower form.
The archive-tag key now concerns the records-side wait, its missing bound, and the tag fetch on another machine (F10).
The parent-check key now concerns the beam ready flag, which sits in the beam worktree and not on the board (F11).
The partial-finish key now concerns the hub-pause and owned-elsewhere refusals being filed as partial finishes (F9).
Round 2 adds one BLOCKING design finding: D2 asks a live writer to write `_lym/move-note.md`, which its role forbids (F8).
