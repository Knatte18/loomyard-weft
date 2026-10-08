---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, check D3 and D5 against the dispatch path: `planglyph.ValidateDispatch` calls `planparser.ValidateFormat`, and `resolvePass` (which emits `delete-before-reference`) is shared by `ValidateFormat`, `Validate` and `ValidateDispatch`. Settle whether the record decides which new checks may run at dispatch and how any plan-gate-only check is kept out, and whether the Goal and the acceptance criteria match that decision."
  - "In `_lyx/discussion/decision-record.md`, check D1 against D7's card order: webster gates each card on `go build ./... && go test ./...`. Settle whether the record says when the test rows for D2's retired containment checks, D3 and D5 start passing, so that no card lands a failing row."
  - "In `_lyx/discussion/decision-record.md`, check whether the record names how a defect the D1 test exposes becomes a card, given the plan is written and gated before any test runs and webster does not add cards."
  - "In `_lyx/discussion/decision-record.md`, check D5's caller-coverage scan against `internal/planparser/deleteorder.go`, whose `LaterDeleteReferences` searches only later cards' Edit regions: settle whether the record states the file set the scan reads and the under-reporting cases (aliased and dot imports) with their bound, per the attack-surface question."
  - "In `_lyx/discussion/decision-record.md`, check D5's re-sign arrow `<glyph> -> <head>`: settle whether the record states what reads the head, what a head that names the right member but misstates the signature lets through, and the arrow's outcome on a non-member `Edit` target."
  - "In `_lyx/discussion/decision-record.md`, check that no edit made to settle the questions above only added content: where text restates another decision or a process fact the plan writer cannot act on, the relocation test applies, extracting any requirement into Decisions or Constraints first."
---

Each entry names a place and a question; none pre-states an answer.
