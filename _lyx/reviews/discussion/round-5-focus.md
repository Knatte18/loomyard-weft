---
round: 5
focus:
  - "Caller coverage in `decision-record.md` (D5): for a `Delete`, a `Rename` Old side and a re-signed `Edit` member, does the coverage direction stated for each kind hold, given that D1 requires every card to leave the suite green and an earlier card cannot write a call against a name or signature that does not exist yet?"
  - "The over-reporting bound in `decision-record.md` (D5 and Open risks): for a package-level member, the reused same-package pattern from `deleteorder.go` is a line-textual whole-word match; does the Bound state what it matches (selectors, struct keys, comments, strings) and how a blocking finding is cleared, or require a tighter pattern?"
  - "The plan-writer note and D5 on where `LaterDeleteReferences` is called: `resolvePass` is shared with `ValidateDispatch`, and D3 and D5 forbid caller coverage at dispatch; do the note and D5 agree on where the new check goes?"
  - "Scope against D6 in `decision-record.md`: do both state the same qualifier for the file-paths-only rubric finding, and do the acceptance criteria cover the arrow case where the same member is re-signed by two cards and the Rename to-side case where the draft handle spells the package self glyph's unit and the Old glyph's unit is another?"
---

Round 4 left two gating design findings, both about D5's reference scan.
The entries ask where to look and which question to settle; they do not restate any prior review.
