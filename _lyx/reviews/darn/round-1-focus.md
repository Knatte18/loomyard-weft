---
round: 1
focus:
  - "internal/loomcli/checklist.go `checklistVisitLine`: does every path that drew the output (row visit list, review block, recipe-unreadable list) now omit it, and does anything else still print a written-file path in the overview?"
  - "internal/loomcli/checklist.go `checklistGateLines`: is the time taken from the gate entry's own first span with a parsable start, with `--:--` when none parses, and is it rendered in UTC?"
  - "`checklistReviewBlock` round grouping: does it still read the round from the visit's output after the display was dropped?"
  - "Docs in the same diff: do `internal/loomcli/doc.go` and the verb's `Long` help match the new line formats, and does docs/overview.md or PATTERN.md need a change?"
  - "Test coverage in `TestRenderChecklist`: do the changed rows pin the new gate time, the `--:--` case and the next-span fallback, without duplicating an existing row?"
  - "Change description claims in `_lyx/landing/summary.md` against the diff."
---

Seeded from the board entry (drop written-file paths, time gate lines in UTC) and the change description.
