```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F8), through the fix-proposal lens.

**LOW — `internal/testcover/mutant.go`, PlanMutants doc line "each operand of a chain of && or of ||, which the mutant deletes by replacing its value, so the operand is still evaluated;"**
- The F6 wording pushes this line to about 123 characters of text.
  By my count it is 39 + 49 + 35 characters across its three clauses.
- The prose rule is under 120 per line, and the review itself asked for line limits to hold.
- Break it at the comma before "so", or move the evaluation note to its own sentence.
- The first line of the new `OperatorDeleteClause` comment is about 119 characters, right at the limit.

**No further findings.**
I built a case against each proposal and none holds:
- **F3 (unfix string map keys):** with a named key type such as `map[Kind]int{"k": 1}`, `lymMutantString("k", id)` infers `string`, so the mutant does not compile.
  The build already drops those through the unbuildable-line path, which costs build rounds.
  It is bounded by the `-e` flag and is not a wrong result.
- **F3, array and struct keys:** a string-literal key cannot occur in an array or struct literal, so the exemption cannot unfix a key that must be constant.
  Non-constant keys also cannot trigger the duplicate-constant-key compile error.
- **F5 (reader propagation):**
  - A variable named in a `FuncLit` parameter or local of another variable's value could add a spurious edge.
    That only attributes extra blocks, and it adds no mutant that does not compile.
  - Composite-literal keys and selector names are held by `markFixed`, so a field named like a package variable adds no edge.
  - The fixpoint cannot loop forever: `addReader` returns false for a key already held, which covers self-references, so each pass adds only new keys.
- **F2 (sort the block indexes in the test):** it cannot hide a missing block, because a missing or extra block changes the slice length.
  `slices.IsSorted` pins the order of the keys separately.
  Reverting `parsePackageFiles` to profile files only would drop the `fixvars.go` mutant and fail that row.
- **F1 (fixture extension):** the new tests were seen failing nothing wrongly and passing in tier 1.
  The `Unique` entries on `TestAllowedChecked` pin the delete-clause and embedded-line operators by name, so a wrong identity value or line index would drop them.
- **F6, F7, F8:** the doc statements match the code.
  `lymMutantDropAnd` and `lymMutantDropOr` do take the already-evaluated operand, and `build.Default` does ignore `GOFLAGS`.

Further tool calls made: none, my context already held every changed file.
