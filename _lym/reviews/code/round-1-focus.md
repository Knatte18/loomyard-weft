---
round: 1
focus:
  - "internal/testcover/mutant.go literal mutants: does every data literal the record names (strings, list members such as `holdFreeProducers`, map entries) get a mutant, and does each mutant stay buildable and avoid mutating non-data literals?"
  - "internal/testcover/mutant.go clause deletion: is each operand of a `&&`/`||` chain deleted, including chains of three or more operands, and is the deleted-operand result valid Go (types, parenthesization, short-circuit meaning)?"
  - "Package-level data mutation: do site line numbers, block attribution and the overlay writer map each mutant to the covering blocks correctly, so a mutant outside any function body is still judged against the right tests?"
  - "Done-when: would `lym gate redundant internal/boardengine` stop listing the five named candidates, given how kills and survivors are decided? Check the guard path in internal/gatecli/redundantmutants.go for regressions."
  - "Test economy in internal/testcover/mutant_test.go and internal/gatecli/redundantmutants_guard_test.go: do the new tests pin distinct behavior, and does each new mutant operator have coverage?"
  - "Claims in internal/testcover/doc.go and internal/gatecli/doc.go against the diff: do the updated descriptions match what the mutator now does?"
---

Seed focus for round 1.
The diff is three commits over `96b20f1c`, most of it in `internal/testcover/mutant.go`.
The decision record asks for two fixes: literal and list-member mutants, and a clause-deletion mutant per boolean operand.
