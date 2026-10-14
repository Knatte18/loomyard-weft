---
round: 1
focus:
  - "D3, D5, D6 in decision-record.md: does the 'writer never edits after validate passes' rule stay consistent with D6 (writer follows a parent correction) and the rejection respawn path, and is there a window between validate passing and D4's Go commit where a `decision add` append can corrupt or lose the writer's files?"
  - "D7 in decision-record.md: it removes the live-board-body source of truth for the Darn-Review and replaces it with the board entry as amended by the record; does the Decision state what the review can now skip or let through (for example an unrecorded correction, or an entry by a non-authoritative author) and what bounds it?"
  - "D9 and D4 in decision-record.md: the new PR-Align landing edge and the Go commit through the fabric are new routing edges; check each Decision states what it can now let through and its bound, and that the bound named is real rather than only asserted."
  - "Acceptance criteria against Decisions D1-D11: does each Decision with an observable behavior have a criterion, and does any criterion assert something no Decision decides?"
  - "Relocation check for decision-record.md: Constraints and Open risks entries that are deliberation narrative or plan-level bookkeeping (docs list, test-budget line, merge-order risks) rather than a requirement the plan writer needs; extract any requirement into Decisions or Constraints before proposing a move to support-log.md."
---

Seed focus only; it narrows the first look and does not cap severity or limit the review to these questions.
No entry above targets the Do-not-flag items: the optional Notes subsection, rejected alternatives in the record, or call-site enumeration.
