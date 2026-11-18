Task: check the correctness of the four fixes (F1 to F4) through the fix-code-correctness lens.

**No findings.**

What I traced, each from the context, with no tool call:

- **F1, `markConstantIntegerExpression` and `isConstantInteger`:**
  - For `(1 << 63) / 2`, the outer `/` has a `ParenExpr` over `1 << 63` and the literal `2`. Every leaf is an INT literal, so every literal is fixed and the plan has no change-constant mutant on that line.
  - `n * 2` and `x + 1` have a non-constant operand, so they are untouched and still mutate.
  - A nested constant sub-expression inside a non-constant outer expression is fixed when it is visited. `Inspect` visits the parent before its children.
  - A local that shadows a package constant is excluded by `locals`. The package-level variable path passes `nil`, which is safe to read.
  - Fixing an `Ident` cannot suppress an embedded-line site, because `isConstantInteger` accepts only identifiers whose helper is `stepHelper`.
  - The new `TestPlanMutants` row (`blockHalf`, want nil) would get sites, and so fail, if the marking were removed. The gate log shows it passing.
- **F2, stand-in and nested subtest:**
  - The five refused lines (13, 18, 26, 27, 29) each hold mutants, so rounds 1 to 4 drop by line.
  - If `wholeFiles` were never passed, build 5 would drop only the line-29 mutants. Build 6 would then reach the real go, which refuses Timeout's mutant, so `Unbuildable` would not be 21.
  - If the cap still failed the check, the exit code would be non-zero.
  - If `wholeFiles` were always set, round 1 would drop everything, round 2 would find nothing to drop, and the call would fail.
  - The subtest reads the map the enclosing subtest built, so running it nested and in sequence is right.
  - The gate log shows it passing.
- **F3, new rows:**
  - Rows 4 and 5 expect `a=~b: c` and `~key:  `. A separator rule that took `: ` first, or had no blank-value fallback, would produce different text and fail.
  - The row ids match the non-blank line order (name 1, note 2, list 3, `a=b: c` 4, `key:  ` 5, plain 6).
- **F4:** the doc now matches `round >= mutantBuildRounds`, with rounds counted from 1.

Calls made: none (0 of 6). The context already held every changed file and the gate log excerpts.
