---
round: 1
focus:
  - "internal/driftengine/judgesnapshot.go `isJudgementPath`: does it exempt exactly the timestamped copies lym's rotation makes (via `shedadapters.ArchivedRunDirSegment`) and nothing a judge could edit, such as other `.yaml` files in `_lym/drift/`; is the code-checkout exemption (same predicate applied to the code side) licensed by the board entry, which names only the records side?"
  - "internal/loomcli/driftwiring.go `loomRecordsDirs`: is `underAnchor(shedrun.UsageDirRel())` the correct anchored path matching where session usage records are actually written (e.g. `src/csharp/NORCE.Models/_lym/usage/`), and does `isLoomRecordsPath` treat it as a directory prefix without exempting sibling names that merely share the string prefix?"
  - "Does the diff leave any other path a judge session writes through lym (beyond usage records and archived judgements) that still halts, or widen the exemption past what the entry asks, hiding real judge edits?"
  - "Tests in judgesnapshot_test.go, driftwiring_test.go and export_test.go: do they cover the new behaviors (archived judgement exempt, non-matching sibling not exempt, usage dir in `loomRecordsDirs`) without duplicating existing coverage; is the export_test.go addition needed?"
  - "doc.go and the `Changes` doc comment: do they match the code as changed, and does the commit message claim anything the diff does not bear out?"
---

Seed focus for the first review round over `cd7fa83..HEAD`; the decision record holds only the Goal, so the amended board entry is the measuring stick.
