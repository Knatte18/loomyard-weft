---
round: 2
focus:
  - "internal/reportengine/landings_test.go exact-join rows and landings.go slug comparison against the pair-opening commit: does a row put a tag's own slug's pair-opening commit below its trailer commit, and another slug's opening commit in the same history, while the expected exact-join mapping stays unchanged?"
  - "internal/reportengine/landings_test.go row whose trailer lies past the first page: does the pair-opening floor land on a later page, so the page-growing walk meets the floor and the mapping stays exact?"
  - "internal/fabricengine/origin_integration_test.go TestAdd origin-record test against add.go's record commit: is the commit subject asserted equal to OriginRecordSubject(slug) and once to the literal subject text, so a changed prefix fails a test?"
---

The focus entries ask the reviewer to settle whether the coverage gaps are closed.
