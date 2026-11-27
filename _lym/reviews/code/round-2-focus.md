---
round: 2
focus:
  - "internal/planparser/latercaller.go, isProductionFunctionDeclaration: which new functions does create-caller-later treat as production code? Check functions under internal/testkit, internal/gitkit and internal/hubforge, and helpers of package-internal _test.go files, against the roots in cmd/lym/testonlycode_linux_test.go. Check that contracts/specs/loom-plan-checks-spec.md and _lym/landing/summary.md state the same skip conditions and limits as the code."
  - "internal/planparser/latercaller.go, checkCreateCallerLater and the caller-side and creator-side card classification: can a later card that creates the caller as a new func be judged correctly, and does the creator-side judgement stay as the answer key needs?"
  - "internal/planparser/latercaller_test.go, TestValidateFormat_CreateCallerLater: is each classification branch pinned by a row that a mutant would fail? Cover the group-type filter, the handle-ref branches, the test-name skip, the choice among several later cards, bare symbols, test glyphs and members, and the finding's Ref and Detail."
  - "contracts/specs/loom-plan-spec.md, the Plan vs. schedule section: does its list of checks that refuse a dependency on a later card match the checks the code runs?"
  - "_lym/landing/summary.md, Merge danger: does the blast radius cover every caller of ValidateFormat, including plan gates and dispatch validation of a run in flight?"
---
