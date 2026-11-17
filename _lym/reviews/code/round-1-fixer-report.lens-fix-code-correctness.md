```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I traced the changed mutator code and each added test against its defect, using only the context I already held.
I made no further tool calls.

**1. MEDIUM — `lymMutantLine` line index is still unpinned.**
- Text: `TestAllowedChecked` in `redundant_integration_test.go`, which asserts only banner line 2, and the `Unique` entry `{cov.go, 17, change embedded line}`.
- Summary: an off-by-one in `lymMutantLine` still passes, so the "wrong line index" half of F1 is not fixed.
- Evidence:
  - With `lines[line]` instead of `lines[line-1]`, the line-1 mutant edits "second" and is killed.
  - The line-2 mutant edits the empty element after the trailing newline and survives.
  - That leaves exactly one kill, listed as `{17, change embedded line}`, which is what the test expects.
  - `Unique` is keyed by file, line and operator, so it cannot tell which mutant was killed.
- Fix: assert line 1 as well as line 2 in the test.
  A correct index then kills both mutants and lists two entries, while the off-by-one kills only one.

**2. LOW — `lymMutantDropAnd` is never switched on.**
- Text: the fixture function `Allowed`, which holds only an `||` chain.
- Summary: the F1 premise names both `lymMutantDropAnd` and `lymMutantDropOr`, but only `lymMutantDropOr` is exercised.
- Evidence: a `lymMutantDropAnd` that returned `false` would pass every test, since only the text of the helper file is checked.
- Note: the `DropOr` identity value is pinned.
  If it returned `true`, the owner clause would be killed too, giving two `delete clause` entries instead of the one expected.

**3. LOW — two new comment lines are at or over the 120-character limit.**
- `PlanMutants` doc, the line "each operand of a chain of && or of ||, which the mutant deletes by replacing its value, so the operand is still evaluated;": about 123 characters.
- The `OperatorDeleteClause` comment, first line: about 120 characters.
- Both need a split at the clause boundary.

**Checked and correct:**
- The `readers` fixpoint terminates, and the `name == variable` case adds nothing.
- `variableValueReads` and the string-key change in `markFixed` are correct.
  A map key that does not compile, such as a named string type, falls under the existing unbuildable-mutant path.
- `SkipObjectResolution` breaks no reader of `ast.Object`, and `embeddedFile` still gets comments from `ParseComments`.
- Each new `TestPlanMutants` row fails with its defect in place:
  - The chain row fails if the same-operator skip is removed, because it would give four sites.
  - The map and array row fails if string keys stay held or the array index is mutated.
  - The `base`/`all` row fails without the propagation.
  - The `fixvars.go` row fails if `parsePackageFiles` parses only the profile's files.
  - The Shadow row fails if the locals skip breaks, because `names` literals would appear.

Tool calls: none; the context already held every changed file and the review.
