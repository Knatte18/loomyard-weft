---
round: 2
focus:
  - "internal/planparser/renumber_test.go TestRenumber and internal/planparser/renumber.go mapsFaithfully: does a test row reach the kept-range clause for an insert, where a bare card range has the inserted card strictly inside it or ends at the insert position, and a boundary row where a range ends just before the insert position stays unchanged and unlisted?"
  - "internal/planparser/renumber_test.go failure-reversal test and applyRenumber: is the write of a card that keeps its number and changes only by a bare card N reference faulted, with the directory restored byte for byte and no journal left?"
  - "internal/planparser/renumber_test.go TestRenumber fixture: does a bare card N reference on a Card Index line or a card heading line stay untouched by the bare scan, with only the Card Index renumbering changing those lines?"
  - "internal/planglyph/index_test.go TestIndex_AppendsPlanChecks and glyphlessFile: do rows cover a missing test file, a test file with a syntax error, and a same-base file in another directory that has symbols, each keeping the edit-test-file finding?"
  - "contracts/specs/loom-plan-spec.md near line 226: does the renumber sentence read clearly, with the antecedent of the leave-as-is-and-report clause unambiguous and the card-id rewrite stated separately?"
---

Focus lists where the round 1 review asked for coverage; each entry is a question to settle against the current diff.
