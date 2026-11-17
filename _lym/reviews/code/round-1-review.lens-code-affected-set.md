```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what reads it and check whether the diff leaves it wrong.

**Affected set per change**

- **`Mutant.Block` became `Mutant.Blocks`, and `Mutant.InBlocks` was added.**
  - Readers are `internal/gatecli/redundantmutants.go:175` and `:304`. Both are updated.
  - The fixtures in `internal/gatecli/redundantmutants_guard_test.go:76` and `:135` are updated.
  - A repo-wide grep for `.Block` finds no other reader of the mutant field. The `.Block` hits in shedengine and stencilstore are unrelated types.
  - Covered.
- **New operator strings (`OperatorDeleteClause`, `OperatorChangeEmbeddedLine`).**
  - Readers are `redundant.go:305` (the output's `redundantMutant.Operator`), `redundantmutants.go:351` (`mutantStop`) and `:490` (the run log). All pass the string through without matching specific values.
  - `redundant_integration_test.go:309-346` asserts only the existing operators.
  - Covered.
- **New sites: clause deletion, constant uses, package-variable literals and embedded lines.**
  - The fixed mutant counts in `internal/gatecli/redundant_integration_test.go:297` (`Mutants != 14`, `Unbuildable != 1`) and `:305` (run counts 8, 29, 16) depend on which sites exist.
  - I checked the `cov/cov.go` fixture at `redundant_integration_test.go:44-88`:
    - It has no `&amp;&amp;` or `||` and no package-level `var`.
    - Its one constant use, `hoardStop` at fixture line 77, sits in Hoard's loop body. No test covers that body (only `Hoard(0)` is called), so it is no target.
    - `time.Second` is a selector, which stays as written.
  - The counts should not change. Covered.
- **The `PlanMutants` contract: it now reads non-block files, parses comments and reads embedded files.**
  - Its one production caller is `redundantmutants.go:381`. It passes `c.pkgDir`, so the embedded-file path resolves.
  - The doc paragraphs at `internal/testcover/doc.go:42-50` and `internal/gatecli/doc.go:1181` and `:1197` are updated.
  - Covered.
- **The doc wording "mutant's block" became "one of the mutant's blocks".**
  - Both sites are updated: `gatecli/doc.go:1197` and `redundantmutants.go:200`.
  - `docs/overview.md:963` says "each mutant of its blocks on", which still holds.
  - Covered.
- **Rule documents.**
  - `pattern/PATTERN-test-speed.md:225` and `:311-321` describe the verb.
  - Line 313, "It mutates the candidates' blocks", is now incomplete: literals of a package-level `var` value and embedded-file lines lie outside any block, and are only attributed to blocks.
  - The bound at line 320, "a mutant changes the package's own code only", still holds.
  - Nothing else in `PATTERN.md` or `pattern/` mentions mutants. `PATTERN-sole-parsers` lists no Go-source or embed format.
- **Declared PATTERN changes.** This round's directive declares none, so there is nothing to land.
- **Commit messages and departures.** None of the three commits in the range has a `Pattern-Departure:` trailer. The first commit's message matches the diff.
- **Stencil wording.** The diff changes no stencil, so this does not apply.

**Findings**

1. **NIT, consistency**, at `pattern/PATTERN-test-speed.md:313`.
   - The background still says `lym gate redundant` "mutates the candidates' blocks".
   - The diff now also mutates literals of package-level `var` values and embedded-file lines. Those sites lie outside every coverage block and are only attributed to the blocks that read them.
   - The sentence is incomplete rather than false.
   - Evidence: `internal/testcover/mutant.go:202-210` (the `PlanMutants` doc) and `pattern/PATTERN-test-speed.md:311-314`.

No other member of the affected set is left wrong.

Tool calls:
1. Grep for operator names, `.Block` readers and the gatecli doc's lines 1175–1200. My context did not have the readers outside the diff or the full doc paragraph.
2. Grep for markdown docs and PATTERN files that mention mutants, and for readers of `.Operator`. My context did not have the docs outside the diff or the output consumers.
3. Grep of `docs/overview.md` and `redundant_integration_test.go` for mutant assertions. My context did not have the integration test's fixed counts.
4. Read of `redundant_integration_test.go:40-189`. I needed the fixture source to tell whether the new sites change the asserted counts.
5. Grep of `pattern/PATTERN-sole-parsers.md`. I needed to check whether a Go-source or embed format has a sole parser the diff bypasses.
