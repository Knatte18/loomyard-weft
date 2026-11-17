```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I named the test row that would fail if it broke.

## Behaviours and the tests that pin them

| Behaviour | Pinned by |
|---|---|
| Each `&amp;&amp;`/`||` operand gets a `delete clause` site | `TestPlanMutants`, row "each operand of a boolean chain is deleted…" (four sites at line 35) |
| Operand of `&amp;&amp;` goes through `lymMutantDropAnd`, operand of `||` through `lymMutantDropOr` | `TestMutantPlan_WriteOverlay`, the expected text of `if a &amp;&amp; b || s == names[0]` |
| A same-operator chain (`a &amp;&amp; b &amp;&amp; c`) adds no site for the inner chain itself (`addClauses` skips it when `inner.Op == expr.Op`) | **none** |
| When the mutant is on, `lymMutantDropAnd` returns true, `lymMutantDropOr` returns false, and `lymMutantLine` puts `~` after the indentation of the given line | **none** |
| A literal in a package-level `var` value is a site, and its block is the block that reads the variable | `TestPlanMutants` row (line 29, `blockPickHead`); `TestMutantPlan_WriteOverlay` (`[]string{lymMutantString("a", 6), …}`) |
| A variable that no target block reads holds no mutant | `TestPlanMutants`, row "a use of a package constant…" (`blockPickTrue`) |
| A site read from several blocks gets every one of them, sorted and deduplicated, and `InBlocks` matches any of them | **none**: every fixture site has exactly one block |
| A multi-name `var a, b = …` gives each literal to the right variable | **none** |
| A composite key, array length, `FuncLit` body or blank `_` name inside a package `var` value stays unmutated | **none** |
| `markFixed` now also fixes identifiers: a field name, a selected name (`x.suffix`), a composite key named like a constant | **none** |
| A use of a package constant whose value is a literal is a site | `TestPlanMutants` row (line 36) |
| A constant whose value is not a literal is not a site | **none** |
| A use of an embedded string is a site for each non-blank line of its file | `TestPlanMutants` row "a use of an embedded string…" (two sites, so the blank line is skipped); `TestMutantPlan_WriteOverlay` (line argument and ID order) |
| `embeddedFile` finds the directive on `gen.Doc` (ungrouped `var`) | the fixture's `//go:embed fix.txt` |
| `embeddedFile` finds it on `value.Doc` (grouped `var`), and refuses a glob, several patterns or a non-string type | **none** |
| An embedded file that cannot be read is an error | **none** |
| A parameter or `:=` local named like a package constant or embedded variable is not a use of it (`fileSites`) | `TestPlanMutants`, row "a parameter or local…" (`blockShadow`) |
| `variableReads` skips a local that shadows a package variable | **none**: the shadowed name `text` has no literals, so a broken skip changes no mutant |
| `parsePackageFiles` also parses non-test files with no coverage block, filtered by `build.Default.MatchFile` | **none**: the fixture has one file |
| At one span, the earlier mutant is the outer call (ties ordered by rank) | `TestMutantPlan_WriteOverlay` (`lymMutantBool(lymMutantDropOr(…))` and `lymMutantLine(lymMutantLine(text, 3, 17), 1, 16)`) |
| `Mutant.Blocks` is reported | the Blocks column of `TestPlanMutants` |
| `gatecli` selects mutants and covering runs through `InBlocks` | `TestRunMutantsRerunsACoveringTestThatTimedOutAsACandidate`, `TestRunMutantsGuardKilledRunIsNoKill` (one-block case only); `TestGateRedundant` end to end |

## Findings

1. **MEDIUM · design** · `internal/testcover/mutant.go:66-107` (`mutantHelpers`): no test runs the new helpers. Nothing checks what `lymMutantDropAnd`, `lymMutantDropOr` or `lymMutantLine` return when switched on.
   - `TestMutantPlan_WriteOverlay` (`internal/testcover/mutant_test.go:303`) checks only that the helper file opens with `package fix` and names `MutantEnv`.
   - `TestGateRedundant` compiles the helper file, but its `cov.go` fixture (`internal/gatecli/redundant_integration_test.go:44-87`) has no `&amp;&amp;`/`||` chain and no embed directive, so no new helper is ever switched on.
   - `TestBuildMutantBinary` (`internal/testcover/runner_test.go:128-160`) runs a fake `go`.
   - So a wrong identity value, or an off-by-one in `lines[line-1]` or the indent split, would pass every test.

2. **MEDIUM · design** · `internal/testcover/mutant.go:637-647` (`addClauses`): the rule that keeps a three-operand same-operator chain at one site per operand is untested. The fixture chain `a &amp;&amp; b || s == names[0]` (`mutant_test.go:52`) mixes operators, so the `inner.Op == expr.Op` branch never runs. The focus names three-operand chains explicitly.

3. **MEDIUM · design** · `internal/testcover/mutant.go:318-352` (`parsePackageFiles`): parsing a file with no coverage block (`os.ReadDir`, `_test.go` skip, `build.Default.MatchFile`) has no test. `planFixture` writes a single `fix.go` that holds blocks (`mutant_test.go:125`). Reverting to "parse only `sources`" passes every row, although the summary claims var-only files are now parsed.

4. **MEDIUM · design** · `internal/testcover/mutant.go:471-503, 546-580` (`markFixed` and the `GenDecl` branch of `fileSites`): no row pins the exclusions.
   - For package `var` values: a composite key, an array length, a `FuncLit` body, a blank `_` name, multi-name attribution.
   - The new identifier fixing: a selected name, a field name, a composite key named like a constant.
   - A constant whose value is not a literal.
   - The fixture's only package variable is `var names = []string{"a", "b"}` (`mutant_test.go:46`), and it holds no composite key, selector or struct literal. A regression here would plan mutants that fail to compile or are not data.

5. **LOW · design** · `internal/testcover/mutant.go:292-298` (block union in `PlanMutants`) and `:134-139` (`InBlocks`): the multi-block case has no test. A variable read from two blocks should give a sorted, deduplicated `Blocks` list, and `InBlocks` should match on any of them. Every expected site in `TestPlanMutants` has a single block (`mutant_test.go:166-231`), and the gatecli guard tests build one-block mutants (`redundantmutants_guard_test.go:76,135`). This is the path that attributes `holdFreeProducers` to every block that reads it.

6. **LOW · design** · `internal/testcover/mutant.go:381-384, 398-418` (`scanPackageDecls` / `embeddedFile`): untested.
   - The documented error for an unreadable embedded file.
   - The `value.Doc` path for a grouped `var`.
   - The refusal of a glob, of several patterns, and of a non-string type.

7. **LOW · design** · `internal/testcover/mutant.go:515-518` (`variableReads`): skipping a shadowing local is not pinned. In `Shadow`, the parameter `text` shadows a variable with no literals, so a broken skip changes no mutant and `TestPlanMutants`' `blockShadow` row still passes.

Tool calls:
1. `grep` for tests that build or run mutant binaries: the manifest did not show whether any test compiles or runs the helpers.
2. `Read` of `internal/gatecli/redundant_integration_test.go:1-300`: to check whether `TestGateRedundant`'s fixture reaches the new operators.
3. `grep` of `internal/testcover/runner_test.go` for helper use: to confirm `TestBuildMutantBinary` does not run real helpers.
