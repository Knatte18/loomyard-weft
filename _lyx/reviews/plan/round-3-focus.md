---
round: 3
focus:
  - "In _lyx/plan/04-forkaudit-card-arm.md and tools/forkaudit/report.jq: settle whether a reader of the printed report, not the jq source, can tell that the card arm's reached-span window (after the card-pack call that lists the span, before the next edit) differs from the other arms' window (before the batch's first edit), wherever the Pack use rows and the 'forks with 3+ parallel calls in the first work turn' row print both arms side by side."
  - "In _lyx/plan/04-forkaudit-card-arm.md, the pack-entries.jq bullet and tools/forkaudit/forkaudit.sh classify_span: settle whether spans from a card-arm fork's second and later card-pack calls, resolved after earlier cards' edits, are graded against a tree that holds them, or against the batch's start sha, and whether a Tests row reaches that case."
  - "In _lyx/plan/04-forkaudit-card-arm.md, the extract.jq bullet and tools/forkaudit/extract.jq ex_net and pre_edit_net: settle whether the card arm's card-pack calls are netted out of the net columns the way the file arm's single pack read is, and whether a Tests row pins the cells."
---

Each entry names a site and a question; none caps severity.
