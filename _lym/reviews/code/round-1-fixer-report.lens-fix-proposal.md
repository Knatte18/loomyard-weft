Task: challenge the proposals the fixer carried out as proposed (F1–F4).

**Findings**

- **LOW — F1, `internal/testcover/doc.go:47-49`.**
  - The package doc still says every integer or rune literal, and every use of a package-level constant holding one, is a change-constant site.
  - The fix now exempts those inside an expression built from integer and rune constants alone, such as `(1 << 63) / 2`.
  - Only the `PlanMutants` doc comment in `mutant.go` records the exemption, so the package doc is stale for a reader.
  - Evidence: the `doc.go` lines above carry no exception.
  - `PlanMutants` states the exemption beside its own list of kept literals.

**Proposals challenged without a finding**

- **F1 shadowing and nesting.** `x / ((1 << 63) / 2)` still exempts the inner constant expression, because the visit reaches it as its own `BinaryExpr`.
- **F1 shadowing and package-level variables.** A local named as a literal constant is not exempted, since `locals` is checked. The package-level variable path passes a nil `locals`, which is safe to read.
- **F1 test.** The `Half` row fails with the defect in place, since `1`, `63` and `2` would each yield a change-constant site.
- **F2 whole-file drop.** The refused lines 13, 18, 26, 27 and 29 each hold mutants, per the sibling test's Unique lists, so rounds 1–4 drop by line only.
  - Build 5 then drops every remaining mutant of `cov.go`, giving 21 unbuildable.
  - A missing `wholeFiles` argument would leave fewer than 21 dropped.
  - Reverting to the old failure at the cap would exit non-zero.
  - Either defect fails the test, and the sixth-build count also checks the cap wiring.
- **F2 stand-in.** The stand-in forwards every other go call to the real go command, so the coverage build and the profile run are unaffected.
- **F2 hard-coded count.** The literal 21 duplicates the sibling assertion's 20 buildable plus 1 unbuildable. It is a perishable tally, but the review asked for the exact count and the gate run passes.
- **F3 separator order.** A first-match-wins implementation would pick `: ` first on `a=b: c` and mark `a=b: ~c`, so the earlier-separator row fails it.
- **F3 blank-value fallback.** The `key:  ` row (colon-space, then blanks only) fails an implementation that marks after the separator.
- **F4.** The reworded constant doc matches `round >= mutantBuildRounds` and the `buildMutants` doc.

**Further calls**
1. `grep` of `internal/testcover/doc.go` and `internal/gatecli/doc.go` for "constant" and "literal", to see whether the package docs state the change-constant rule the fix narrowed. Neither file was in my context.
