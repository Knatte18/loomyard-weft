---
round: 1
focus:
  - "internal/testcover/mutant.go: does the `(c + 1>>lymMutantStep(id))` rewrite keep the type of every integer or rune constant in every context it is applied to, and are constants it cannot keep (untyped in a constant expression, array length, case label, iota, shift operand) skipped?"
  - "internal/testcover/mutant.go and internal/gatecli/redundantmutants.go: does UnbuildableMutants blame by line, then file, then every mutant left without dropping buildable mutants wrongly, and can the drop rounds loop or end with a failure when the package builds unmutated?"
  - "internal/testcover/testfuncs.go: does ScanTestFuncs, which now reads only the default build context's test files, still map every test a Linux run can execute, and does a skipped or unrunnable test no longer land in failed_tests?"
  - "Embedded key-value mutant in mutant.go: does it mark the value and not the key for the first `: ` or `=` with text after it, and fall back to the indent for other lines, as the commit message states?"
  - "Scope against the decision record's Fix section: does the diff make all three fixes, and is any change in it (such as the map fixture's build tag drop) unlicensed by the record?"
  - "Test economy: check the redundancy report's not-judged tests (TestMutantBinary_*, TestBuildMutantBinary, TestGateTestMapLinkedPackages) against tests that assert the same behavior, and whether the changed assertions in mutant_test.go, runner_test.go and redundant_integration_test.go still check what their names state."
  - "Commit messages and docs: do figures or claims in them, and the doc.go changes in gatecli and testcover, match the code as the diff leaves it?"
---

Seed focus for round 1, drawn from the board entry under `## Goal` in the decision record; the record's other sections are empty.
