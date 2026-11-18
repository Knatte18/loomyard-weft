# Card 13 — line-mutant-planner

**Create:**
- `plan:internal/testcover#PlanLineMutants` -> `func PlanLineMutants(dir, importPath string, profile []ProfileBlock, changed map[string][]int, covered BlockSet) (*MutantPlan, []BlockKey, error)`
- `plan:internal/testcover_test#TestPlanLineMutants` -> `func TestPlanLineMutants(t *testing.T)`
- `plan:internal/testcover_test#lineMutantFixture` -> `func lineMutantFixture(t *testing.T) string`

**Edit:**
- `internal/testcover#fileSites`

**Extends:**
- `internal/testcover/doc.go`

**Uses:**
- `internal/testcover#PlanMutants`
- `internal/testcover#MutantPlan`
- `internal/testcover#Mutant`
- `internal/testcover#KeyBlocks`
- `internal/testcover_test#planFixture`
- `internal/testcover_test#TestPlanMutants`

**Intent:**
Add D9's changed-line mutant planner to `internal/testcover`.

`PlanLineMutants(dir, importPath, profile, changed, covered)` places mutants only on the lines a diff adds or changes, in blocks a test covers:
- `profile` is the package's block table from its coverage binary (`Toolchain.ProfileBlocks`), keyed with `KeyBlocks` as `PlanMutants` keys it;
  `changed` maps a file name of the package to its changed line numbers (card 1's `testreach.ChangedLines`, which the caller computes);
  `covered` holds the rendered keys of the blocks a fresh map's measured tests cover.
- A block holding a changed line that no key of `covered` names gets no mutant; it is returned in the second result, as an uncovered changed block.
- Operators, in this order: a negated condition or comparison, a deleted operand of `&&` or `||`, a dropped statement and a shifted integer constant (or rune).
  A string constant, an embedded file line and a dropped call into `internal/logger` get no mutant; the logger's import path is one named constant.
- At most one mutant per changed line, the first site the operator order gives; a site counts for the line its start lies on.
- The plan is a `MutantPlan` like `PlanMutants`', so `WriteOverlay`, `BuildMutantBinary`, `UnbuildableMutants` and `MutantEnv` serve it unchanged.
  The cap of D9 (`mutation_max_mutants`) is the caller's, across packages; each mutant's `Blocks` names its block, whose `Func` lets the caller spread the cap over the changed functions.

`fileSites` keeps its signature: each site it returns for a dropped call statement gains the import path of the callee's package, read from the file's imports (empty for a local or method call).
`PlanLineMutants` reads that field to pass over a call into `internal/logger`; `PlanMutants` ignores it, so its sites and order stay as they are.

`lineMutantFixture` writes a package of its own into `t.TempDir()` and returns its directory, leaving `planFixture` and the constants `TestPlanMutants` pins unchanged:
a file importing `internal/logger` with a call into it, a condition and an integer constant on one line, a string constant, an embedded file and its use.
`fileSites` parses without type checking, so the import path is recognised in a temp directory.

`TestPlanLineMutants` (tier0) is one table over `lineMutantFixture`'s files:
mutants land only on changed lines of covered blocks, at most one per line, the first by operator order (a line holding a condition and a constant gets the negation);
a changed line holding only a string constant, a use of an embedded file and a `logger` call gets none;
a changed line in an uncovered block gets no mutant and its block comes back uncovered; an unchanged covered line gets none;
the plan's overlay builds through `WriteOverlay` as `TestMutantPlan_WriteOverlay` does.

`testcover/doc.go`'s mutation paragraph gains the changed-line planner and its operator set.

**ImpactSummary:** A new planner beside `PlanMutants` sharing its site scan; `fileSites`' call sites gain a callee field that `PlanMutants` ignores, with no signature change, so its plans and tests are unchanged.
