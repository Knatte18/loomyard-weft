---
round: 2
focus:
  - "internal/planparser/validate.go checkRenameLeavesTest: does a test ref in a Go package's external test unit (`<dir>_test`) count as the same unit as a renamed symbol in `<dir>`, and does a test in an unrelated unit still draw no finding?"
  - "internal/websterengine/recordbatch.go record-batch hand-back (`HandBackRemovedTargets: report.Status == ReportStatusOK`): do the tests in recordbatch_git_test.go show that a FAILED report and an exact quarry rename draw no hand-back?"
---

Round 2 re-reviews the fix to the external-test-package gap and the added tests.
