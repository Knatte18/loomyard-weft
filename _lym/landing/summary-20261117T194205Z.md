# gate redundant: mutate package-level data and delete boolean clauses

`lym gate redundant` listed tests as redundant when the only check they add pins data or a single clause.
The check of five `internal/boardengine` candidates (`docs/research/speed/redundant-candidates-check-2026-11-17/report.md`) found every one false.
The mutation check never changed a package-level value, and it never deleted one operand of a boolean chain.
This change adds both to `testcover.PlanMutants`.

## New mutants

- **Delete clause** (`OperatorDeleteClause`): each operand of a `&&` or `||` chain in a function body passes through a helper.
  When that mutant is on, the helper returns the chain's identity: `true` under `&&`, `false` under `||`.
  Negating a clause such as `before.Kind != now.Kind` in `checkRunLocks` breaks every accepted row of the cover.
  Deleting it fails only the test that feeds a kind change alone.
- **Package-level variable literals**: each integer, rune or string literal in a package-level `var` value is a site.
  Composite keys and array lengths stay.
  This reaches the members of `holdFreeProducers` and the entries of a map literal.
- **Uses of package-level constants**: each use in a function body of a constant whose value is a literal is a site, changed as that literal would be.
  This reaches `raddleStoreDirName` in the `git checkout` path that `TestReadRaddleStore` pins exactly.
- **Embedded lines** (`OperatorChangeEmbeddedLine`): a use of a string variable that a `//go:embed` directive fills from one file is a site for each non-blank line of that file.
  When that mutant is on, the line gets a `~` after its indentation, so a YAML key or value changes while the file still parses.
  This reaches `template.yaml` behind `ConfigTemplate`.

A local or parameter named after a package-level constant or variable shadows it in its whole function, which then holds no use of that name.
This errs toward fewer mutants without type information, and it avoids the deprecated `ast.Object` resolution.
Files with no coverage block, such as a file holding only `var` declarations, are now parsed too, picked by `build.Default.MatchFile`.

## Blocks of a mutant

`Mutant.Block` becomes `Mutant.Blocks`.
A literal of a package-level variable's value lies in no coverage block, so it belongs to every block that names the variable.
It is planned when one of those blocks is a target.
`Mutant.InBlocks` answers whether a test's block set reaches the mutant.
`internal/gatecli`'s mutation check uses it to pick a candidate's mutants and the covering tests that run one.

## Nesting on one site

A comparison operand of a chain is now both a negated comparison and a deleted clause on the same span.
`applyMutants` used to emit both closings in plan order, which gave each helper the other mutant's ID.
Ties now open in plan order and close in reverse, so the earlier mutant is the outer call.

## Design decisions

- The literal fix adds mutants rather than listing a candidate only when its covers assert every literal.
  Adding mutants keeps one verdict rule: a unique kill refutes.
- A mutant that does not compile in a typed context is left to the existing unbuildable-drop rounds, as literal mutants already were.
  An example is an untyped constant passed where a named type is needed.
- The `Done when` run of `lym gate redundant internal/boardengine` is not part of this change's tests.
  It needs a coverage map and a `lym` built from this change, which takes about 22 minutes.
  The new sites cover each of the five candidates' unique checks; the run after landing confirms the lists.

## Tests

`TestPlanMutants` gains rows for clause deletion, a package variable's literals and their block, a constant use, embedded lines and a shadowing parameter.
`TestMutantPlan_WriteOverlay` pins the mutated text, including two mutants nested on one span.
