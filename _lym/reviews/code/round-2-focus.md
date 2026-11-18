---
round: 2
focus:
  - "internal/testcover/mutant.go, `fileSites` and `closing`: does the change-constant rewrite `(c + 1>>lymMutantStep(id))` alter the value of an untyped constant expression, such as `(1 << 63) / 2` in a uint64 function, when every mutant is off?"
  - "internal/gatecli/redundantmutants.go, `buildMutants` and the `mutantBuildRounds` loop: does a test drive the loop to the round where `round >= mutantBuildRounds` and assert the call finishes with the whole-file drop's Unbuildable count, rather than failing at the cap?"
  - "internal/testcover/mutant.go `lymMutantLine` and TestMutantBinary_ChangesTheValueOfAnEmbeddedKeyValueLine: do the rows cover the earlier of `: ` and `=` winning, and the fallback to the indent when only blanks follow a matched separator?"
  - "internal/gatecli/redundantmutants.go, the `mutantBuildRounds` doc against the loop starting at `round := 1`: does the doc state the build from which a refused build drops whole files?"
---
