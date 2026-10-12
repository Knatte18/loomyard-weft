---
round: 3
focus:
  - "Card `_lyx/plan/21-orch-pause.md`, `pauseTick`: settle whether a watcher that has been running when a hub pause entry is written, with `--grace 0` or with `poll_interval_ms` at least the grace, still interrupts a busy orch at the entry's `GraceEnd`, and whether the rule for an entry already past `GraceEnd` keys on the watcher's own start rather than on the entry's first read."
  - "Card `_lyx/plan/21-orch-pause.md`: settle whether an orch started between an entry's `PausedAt` and `GraceEnd` is left uninterrupted, as the decision record's table for `lyx orch start` requires, and whether a test row reaches that case."
  - "Card `_lyx/plan/23-pause-cli.md`: settle whether the sandbox scenario number it adds to `tools/sandbox/SANDBOX-CORE-SUITE.md` is unused in that file's scenario headings and its Session log format block."
  - "Card `_lyx/plan/17-batten-pause-reader.md`: settle whether some named test reaches `battenCLI.wire`'s `pauserecord.SetGateSlug` call, so that dropping the call fails a test."
---

Focus targets the places any round-2 fix will touch.
