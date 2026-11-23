---
round: 1
focus:
  - "internal/gatecli/redfirstdotnet.go judgeDotnetRedFirst and projectCompileError: does a test that fails to compile on the merge base because it reads a member the change adds count as red with that reason, and does the gate halt only for a reason the writer can act on?"
  - "internal/gatecli/redfirstdotnet.go run, byProject, nearestProject, filterTerms, failedNameMatches: does each changed test run by its test project and fully qualified name in a scratch checkout of the merge base, and is red judged on the base and green at the head as for Go?"
  - "internal/gatecli/redfirstdotnet.go keepHeadProjectFiles: which files does it carry over from the head, and could a wrong set make a base run pass or fail for a reason other than the test? The mutation survivor at line 212 is a candidate."
  - "internal/gatecli/redfirstdotnet.go filterTerms: the mutation survivor at line 327 drops a statement; settle whether a filter term can be duplicated, unsorted or empty, and whether a test could assert it."
  - "internal/dotnetimpact/source.go and changedtests.go: does the parser select exactly the test methods the diff adds or changes, including nested types, attributes spanning lines and multi-statement members? Check the survivors at lines 118, 233 and 425 against the tests that reach them."
  - "internal/gatecli/redfirst.go and tiers.go: do the shared red-first changes keep the Go path's behavior unchanged, including the verdict exit and the no-test case?"
  - "internal/gitrepo/blobread.go, internal/fabricengine/codereads.go, internal/gateslot/runlog.go: do the new commit-based reads and TierFailureNames behave correctly on failure paths such as a missing blob, an absent commit or a missing run log?"
  - "contracts/recipes/darn-recipe.yaml, contracts/stencils/darn/darn-template-write.md, docs/overview.md and the package doc.go files: do the wording changes match the board entry, and do they drop or weaken a rule the entry did not ask to change?"
  - "Commit messages in the range: check each Root-Cause, Loosened-Check and Pattern-Departure trailer, and any claim of agreement or improvement lacking before and after figures."
  - "Test economy: TestJudgeDotnetRedFirst_ReadsTheRunsExitCodeFailureReportAndCompileErrors is listed as contained in TestDotnetRedFirstCallRun_RunsTheChangedTestMethodsInACheckoutOfTheBase; compare their assertions before filing anything."
---

The decision record holds only the board entry under `## Goal`, so the entry's four expectations are the measuring stick.
The focus entries follow those four expectations first, then the shared code the change touches, then repository rules and test economy.
