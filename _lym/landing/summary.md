# testcover: mutate data literals and delete boolean-chain operands

`lym gate redundant` listed tests as redundant when they alone pin a data literal or one operand of a `&&`/`||` chain.
The check of five `boardengine` candidates (`docs/research/speed/redundant-candidates-check-2026-11-17/report.md`) found all five false.
Its mutants only changed conditions and operators, so a test that alone asserts a string, a list member, a map entry or one clause had no mutant to kill.
The fix is in the mutant code of `internal/testcover`.
The candidates were `TestConfigTemplate_ResolvesToDefaults`, `TestRunHolds`, `TestRunLock_BeamKindAndBeamChange`,
`TestRunLock_SketchExemptionIsTheWholeNetChange` and `TestReadRaddleStore`.

## What changes

- A new operator, `OperatorDeleteClause`, replaces each operand of a `&&` or `||` chain with the value that ignores it:
  an operand of `&&` turns true and one of `||` turns false.
  The operand is still evaluated and only its value is replaced, so its side effects stay.
- The constant-changing operator now reaches data literals, including a string used as a map key.
  It also changes a use of a package-level constant holding a literal.
- A literal in a package-level variable's value is now a site.
  It belongs to each block that names the variable, and to each block that names a variable whose value names it, however many variables lie between.
- At a use of a string variable that an `//go:embed` directive fills from one file,
  one mutant per non-blank line of that file changes that line.
- `Mutant.Block` becomes `Mutant.Blocks`, and `Mutant.InBlocks` reports whether a run of the given blocks reaches the mutant.
  `internal/gatecli` uses it in place of the single-block lookup, both when it picks a candidate's mutants and when it picks the covering tests to run with a mutant on.
  `doc.go` of both packages states the new rule.

## Design decisions

- A package-level variable has no block of its own, so its literals are attributed to every block that reads it.
  A mutant of such a literal is judged by the tests covering any of those blocks.
- Composite-literal keys stay fixed except string literals, which only a map key can be.
  Literals of a constant declaration, an array length and a field tag stay, since they must be constant.
- Package parsing reads the files the default build context takes.
  A build tag set through `GOFLAGS` is not read, so a file gated by one may be skipped or its alternative scanned.
- An identifier a function declares itself shadows a package-level name there, so a use of that local is never counted as a use of the package-level one.

## To check by hand

- `lym gate redundant internal/boardengine` was not run to confirm it no longer lists the five candidates above.
  The `internal/gatecli` redundant integration test was extended to drive the new mutants.
- Three review findings were left open:
  a file with no coverage blocks, and a mutant with several blocks, are only partly pinned by tests in `internal/testcover/mutant_test.go`;
  the string keys of a map literal are mutated, and the test that pins this may not cover every key shape;
  and the mutant helpers are driven through the redundant fixture rather than switched on directly in a unit test.
