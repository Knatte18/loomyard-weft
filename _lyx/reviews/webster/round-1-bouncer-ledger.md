---
round: 1
ledger:
  - key: retire-legacy-drops-legacy-done-before-deletion
    rounds: [1]
    status: resolved
  - key: stale-doc-comments-after-board-json-switch
    rounds: [1]
    status: resolved
  - key: render-compact-pads-by-bytes
    rounds: [1]
    status: resolved
  - key: new-comments-break-line-break-convention
    rounds: [1]
    status: resolved
  - key: promote-note-reports-note-not-found
    rounds: [1]
    status: resolved
---

This is round 1, so there was no earlier ledger.
The reviewer approved the ten-card diff with five findings, and the fixer fixed each one in its own commit.

The one MEDIUM finding was about crash safety in `RetireLegacy`.
It dropped `legacy_done` in the save that came before the legacy files were deleted, so a crash or a failed deletion would re-fold old done marks.
`RetireLegacy` now keeps `legacy_done` until the deletion succeeds and then saves a second time, and a regression test covers it.
The other four were documentation and polish: stale doc comments, byte-based padding in the compact listing, comments that broke the Comment Line-Break Convention, and the error text in `PromoteNote`.

For the Comment Line-Break Convention, the settled reading is this: legacy comments moved unchanged, and one-word substitutions inside a legacy fixed-wrapped comment, keep their wrap.
Rewrapping them would touch lines the convention says to leave alone.

The verify block is green on `33a45197f`, and all entries are resolved.
