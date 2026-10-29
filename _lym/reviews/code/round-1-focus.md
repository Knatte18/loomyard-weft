---
round: 1
focus:
  - "internal/loomshed/verifygate_integration_test.go, new subtest: do the fixture helpers it calls (`f.runGate`, `f.diffGate`, `baseOf`, `firstPass`, `notScoped`, `toldCommand`) exist in scope at that point with the signatures used, and does the darn form read the same state the neighbouring subtests read?"
  - "Same subtest: does the early `return` on failure match how the neighbouring subtests gate later steps, and does the next subtest's clean rewrite of `a/a_test.go` leave no misplaced marker for later steps?"
  - "Same subtest: does the committed file put `//lym:guard` on line 5 above a non-test declaration, so `a/a_test.go:5` and 'directly above' match the refusal text printed by `internal/gatecli/cli.go`?"
  - "Scope against the board entry: the diff must be test-only, with no skip, and cover both the loom gate and the darn gate."
---

Single-file test addition; the focus is on whether the subtest fits the round scenario's shared state and asserts what the board entry names.
