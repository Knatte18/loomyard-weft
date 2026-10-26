---
round: 1
focus:
  - "internal/loomcli/checklist.go: check the box precedence (current producer, then visited, then skipped, then not reached) for a row with both a skip and a visit entry, and for a row that is the current producer but also has skip entries."
  - "internal/loomcli/checklist.go: check the review segment heading box for a segment with no visit and a skipped row, including a segment with some rows skipped and others never reached, against the record's single-heading-line criterion."
  - "internal/loomcli/checklist.go: check that skipped detection reads only OutcomeSkipped history entries through shedengine.Status, with no new parsing, and that a row skipped by its entry gate is the only kind drawn [-]."
  - "internal/loomcli/checklist.go and the repo: check that every other place that reads or documents the old halted [-] glyph (other views, docs, stencils, tests) follows the move to [=]."
  - "internal/loomcli/checklist_test.go and doc.go: check that tests cover each acceptance criterion without duplicating existing rows, and that the doc.go checklist paragraph matches the new boxes."
---

Seed focus from the board entry and the decision record's acceptance criteria.
