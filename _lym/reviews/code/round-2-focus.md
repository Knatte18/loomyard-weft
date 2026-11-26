---
round: 2
focus:
  - "internal/discussionparser/decisionids.go, continuesItem: for each kind of unindented line under a bullet (a line opening with # or < or >, a thematic break of -, * or _, a list marker, a fence), does the code end the bullet exactly where markdown ends the paragraph, and do its doc comment, SectionBullets' doc comment in partsplice.go and contracts/specs/loom-discussion-spec.md state the same rule?"
  - "internal/discussionparser/partsplice_test.go, TestSectionBullets and TestSplicePart: would a test fail if any one ending condition of continuesItem were removed or widened, does a splice read-back cover a criterion with an unindented continuation line, and does the test's doc comment name what its rows pin?"
  - "_lym/landing/summary.md, Merge danger: do the blast radius and the door call cover every reader of the changed bullet text, including raddleparser's coverage check, SplicePart's part records, CheckDecisionIDs and a binary on the base rule reading the same record?"
  - "contracts/specs/loom-discussion-spec.md, the paragraph holding the sentences on where a bullet ends: does each sentence there still follow from the one before it, in particular the one on D1 not being met by D10?"
---
