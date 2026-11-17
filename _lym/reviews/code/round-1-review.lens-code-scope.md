```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped each change in the diff to the board entry's goal points; the record has no `Added after Discussion` entries.

**Decisions (from the board entry's `## Goal`):**
- **G1** — "No mutant changes a data literal": add literal and list-member mutants. The candidates are `TestConfigTemplate_ResolvesToDefaults` and `TestRunHolds`/`holdFreeProducers`.
- **G2** — "No mutant deletes one operand of a boolean chain": a clause-deletion mutant for each operand. The candidates are `TestRunLock_BeamKindAndBeamChange`, `TestRunLock_SketchExemptionIsTheWholeNetChange` and `TestReadRaddleStore`.
- **G3** — "Done when": `lym gate redundant internal/boardengine` no longer lists the five candidates.

**Changes and what they trace to:**
- **C1, clause deletion:** `OperatorDeleteClause`, `lymMutantDropAnd`/`lymMutantDropOr` in `mutantHelpers`, `addClauses`, and the `*ast.BinaryExpr` case in `fileSites` (mutant.go:41, 66–78, 598–602, 634–647). Traces to G2. It reaches `checkRunLocks` runlock.go:180 `before.Kind != now.Kind || … before.Beam != now.Beam`. It also reaches `endsAsSketch` sketch.go:81–84, whose `now.Status == nil` operand the Sketch candidate's kept-status row feeds.
- **C2, package-level variable literals:** the `*ast.GenDecl` VAR case in `fileSites`, `site.variables`, `variableReads`, `readers`, `Mutant.Blocks` and `InBlocks` (mutant.go:119–139, 272–298, 505–525, 548–581). Traces to G1, since it reaches `holdFreeProducers` (runlock.go:44).
- **C3, uses of package-level constants:** `packageDecls.constants`, the `*ast.Ident` case and `localNames` shadowing (mutant.go:356–376, 420–456, 607–613). Traces to G1 (string literal data) and G3. It reaches `raddleStoreDirName` at raddlestore.go:150, in the `git checkout` path that `TestReadRaddleStore` pins.
- **C4, embedded lines:** `OperatorChangeEmbeddedLine`, `lymMutantLine`, `embedDirective`, `embeddedFile` and `packageDecls.embedded` (mutant.go:34–35, 98–107, 377–418, 614–616). Traces to G1 and G3. `TestConfigTemplate_ResolvesToDefaults`'s unique assertions (`design_prefix`, the type descriptions, `labels: {}`) are values in `template.yaml` behind `//go:embed` (template.go:10). This is the only route a mutant has to them, so it is in scope.
- **C5, nesting order at one span** (`applyMutants` rank, mutant.go:715–748). Forced by C1: a comparison operand of a chain is now both a negated comparison and a deleted clause.
- **C6, parsing files with no coverage block** (`parsePackageFiles` with `build.Default.MatchFile`, mutant.go:315–352). Forced by C2 and C3, because a file holding only `var`/`const` declarations has no block.
- **C7, gatecli callers moved from `Block` to `InBlocks`, plus the doc lines** (redundantmutants.go:175, 200, 304; gatecli/doc.go:1197; guard_test.go:76, 135). Forced by C2's change of the field type.
- **C8, testcover/doc.go:45–48 operator list.** Forced by C1–C4.
- **C9, mutant_test.go fixture rows and the overlay expectation.** Forced by C1–C5.
- **C10, commits 99417198 and 7958e0f2** (comment line breaks in mutant.go). Forced by the comment lint on C1–C4's comments.

**Decisions and the changes that build them:**
- **G1:** C2, C3 and C4.
- **G2:** C1, with C5 as the change it forces.
- **G3:** C1–C4 between them give each of the five candidates a site that only the candidate feeds.
  - `holdFreeProducers` members.
  - Embedded `template.yaml` lines.
  - The `Kind`/`Beam` clauses.
  - The `now.Status == nil` clause.
  - The `raddleStoreDirName` use at raddlestore.go:150.

  The diff does not show the run itself. summary.md says so ("not part of this change's tests … the run after landing confirms the lists"). The change exists, so this is not a scope gap, but G3 is still unverified. One residual is for the handler or the correctness lens: `TestReadRaddleStore`'s decoded `Hash`/`Project` come through struct tags, which `markFixed` leaves alone (mutant.go:484–490). So that candidate's unique kill rests on the line-150 constant use alone.

Findings: none. Every change traces to G1, G2 or G3 or is forced by one that does, and every decision has changes that build it. No commit carries a `Pattern-Departure:` trailer, and none is needed.

Tool calls:
1. `grep` of `endsAsSketch` (internal/boardengine/sketch.go:80–85) and of `TestSlugRule_RaddleStore`'s assertions. I needed to confirm that C1 reaches the Sketch candidate's unique input; the manifest held neither.
