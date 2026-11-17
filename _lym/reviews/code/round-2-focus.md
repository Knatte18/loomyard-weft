---
round: 2
focus:
  - "internal/testcover/mutant.go helpers lymMutantDropAnd, lymMutantDropOr and lymMutantLine: does a test turn each on and observe a killed delete-clause or change-embedded-line mutant, through the TestGateRedundant fixture in internal/gatecli/redundant_integration_test.go?"
  - "internal/testcover/mutant.go parsePackageFiles and the reader-block union in PlanMutants: does a test cover a Go file with no coverage block and a package variable read by several blocks, with InBlocks matching on a later block?"
  - "internal/testcover/mutant.go markFixed, KeyValueExpr case: are string-literal keys of a map literal mutated, and does the PlanMutants doc comment state what the code does for composite keys?"
  - "internal/testcover/mutant.go variableReads and the operand evaluation of lymMutantDropAnd and lymMutantDropOr: are variable-to-variable reads and the still-evaluated deleted operand handled or stated in the doc comments?"
---
