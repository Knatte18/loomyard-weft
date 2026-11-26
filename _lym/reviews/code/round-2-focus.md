---
round: 2
focus:
  - "contracts/stencils/drift/drift-template-judge.md, Step 2 file rules, against the halt text in internal/driftengine/loosened.go and the package text in internal/driftengine/doc.go: the halt names committing the dropped checks in a test as a way forward. Following the judge's instructions to the letter on a range where one file loses checks and another gains them, with no Loosened-Check trailer, does the halt clear? Does the rule still require a trailer for a widened or dropped check with no counterpart?"
  - "internal/driftengine/loosened.go (listFiles and the range ruling key) with TestDriftVerify in internal/driftengine/verify_test.go: do the tests fail if a stored ruling with no uncaused check stops being reused across a change elsewhere in the range, and if the range key stops covering the content of a file already in the range? Is every input of the range key able to change which ruling is reused?"
  - "The statements of when a file is judged again and which files the judge sees: the halt text and comments in internal/driftengine/loosened.go, the checkAndJudge doc comment in internal/driftengine/verify.go, the RangeKey comment in internal/driftengine/records.go, docs/overview.md on the uncaused-check halt and Drift-Recheck, and _lym/landing/summary.md. Does each match what the code does?"
---
