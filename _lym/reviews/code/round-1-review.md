---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testcover/mutant.go:66
    summary: No test switches on lymMutantDropAnd, lymMutantDropOr or lymMutantLine, so a wrong identity value or a wrong line index passes every test
    fix: In internal/gatecli/redundant_integration_test.go, extend the cov/cov.go fixture of TestGateRedundant with a function holding a two-operand && or || chain and a //go:embed string variable read by a covered block. Add a test of that fixture whose input is decided by one clause and one that asserts one embedded line, so a delete-clause and a change-embedded-line mutant are each killed. Update the asserted mutant and run counts and assert the kills of those two mutants by operator. Add no per-helper test.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testcover/mutant_test.go:122
    summary: Parsing a Go file with no coverage block and attributing a package variable to several reader blocks are both untested; every fixture site has one block and the fixture is one file
    fix: In planFixture, also write a second non-test file, such as fixvars.go, that declares only a package-level var holding a string literal. Read that var in two blocks of mutantSource, such as Classify's tail and Pick's tail. In TestPlanMutants add a row that targets only the later of those blocks and expects the var's literal mutant with File fixvars.go and Blocks holding both block indexes in ascending order. Relax the per-mutant File check so it accepts the new file.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:482
    summary: markFixed holds every composite-literal key, so string keys of a map literal, the data of a set-like map, never get a mutant, and the PlanMutants doc says such a key must be constant
    fix: "In markFixed's *ast.KeyValueExpr case, leave a key that is a *ast.BasicLit of kind token.STRING unfixed, since only a map key can be a string literal; keep holding every other key. Change the PlanMutants doc line at mutant.go:211 to say that a composite key stays unless it is a string literal. Add a package var such as map[string]bool{\"k\": true} to the fixture and expect its key and value mutants in the TestPlanMutants row that reads it."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testcover/mutant_test.go:52
    summary: TestPlanMutants has no row for a same-operator chain of three operands, for a composite key or blank name inside a package var value, or for a local that shadows a package var holding literals
    fix: "Extend mutantSource and TestPlanMutants. Give Pick or a new function a chain a && b && c and expect exactly three delete-clause sites on its line. Give the package var a keyed element, such as []string{0: \"a\"}, and expect no mutant on its key. Make the Shadow function declare a local named names, and expect no mutant of the package var names from the blockShadow row."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:505
    summary: variableReads counts only reads in function bodies, so a package var read only by another package var's initializer gets no reader block and its literals are never mutated
    fix: In PlanMutants, after readers is built, give each package var the reader blocks of every package var whose value names it, repeating until no set grows. Find those names by inspecting each package var value for identifiers that decls.variables holds. Add a fixture var read only through another var's value and expect its literal under that var's reader block.
  - id: F6
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/testcover/mutant.go:66
    summary: A deleted operand is still evaluated, because lymMutantDropAnd and lymMutantDropOr take the operand's value, so an operand with a side effect keeps that side effect under the mutant
    fix: State this in the PlanMutants doc and in the OperatorDeleteClause comment, saying the operand is still evaluated and only its value is replaced. A wrapper that skips the evaluation would break the typing of a named bool operand.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:329
    summary: parsePackageFiles picks block-less files with build.Default.MatchFile, which ignores build tags passed through GOFLAGS in the holder environment the coverage and mutant builds run under
    fix: State the bound in the parsePackageFiles doc comment, saying that a file is picked by the default build context and that tags set through GOFLAGS are not read. Alternatively, pass the -tags value parsed from the toolchain environment's GOFLAGS into a copy of build.Default's BuildTags before calling MatchFile.
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/testcover/mutant.go:345
    summary: parser.ParseComments replaced parser.SkipObjectResolution instead of joining it, so every file now runs the deprecated object resolution, which the summary says the change avoids
    fix: At mutant.go:345, pass parser.ParseComments|parser.SkipObjectResolution to parser.ParseFile.
---

### [MEDIUM] The new helpers are never switched on in a test

**Location:** internal/testcover/mutant.go:66

**Issue:** `TestMutantPlan_WriteOverlay` (`internal/testcover/mutant_test.go:299-305`) checks only that the helper file opens with `package fix` and names `MutantEnv`.
The `TestGateRedundant` fixture compiles the helpers, but it holds no `&&`/`||` chain and no embed directive, so the new helpers are never turned on.
These helpers carry the whole fix.
If `lymMutantDropAnd` returned `false`, or `lymMutantLine` indexed the wrong line, every candidate would silently stay listed, and nothing would fail.
The assistant's finding on the same gap is merged here, at MEDIUM: no rule of the repository makes it blocking.

### [MEDIUM] No test parses a block-less file or attributes a variable to several blocks

**Location:** internal/testcover/mutant_test.go:122

**Issue:** `planFixture` writes one Go file that holds blocks.
Reverting `parsePackageFiles` to parse only the profile's files passes every test, although the summary claims that files holding only `var` declarations are now parsed.
Every expected site in `TestPlanMutants` has a single block, and the gatecli guard tests build one-block mutants.
The union, sort and dedup of reader blocks (`mutant.go:292-298`) and `InBlocks` matching on a later block are therefore unpinned.
That union is the path that attributes `holdFreeProducers` to every block reading it.

### [MEDIUM] Map-literal string keys are held fixed

**Location:** internal/testcover/mutant.go:482

**Issue:** `markFixed` holds every `KeyValueExpr` key.
A struct key is an identifier and an array or slice index must be a constant integer, so a string-literal key can only be a map key, and a map key need not be constant.
The board entry names "map entry" among the data no mutant changes.
A set kept as `map[string]bool{"Finalize": true}` still gets no mutant of its members, and the PlanMutants doc's "since it must be constant" is false for these keys.

### [LOW] Rows missing for several branches of the mutator

**Location:** internal/testcover/mutant_test.go:52

**Issue:** The fixture chain `a && b || s == names[0]` mixes its operators, so the `inner.Op == expr.Op` skip in `addClauses` never runs.
The focus asks about three-operand chains; tracing `a && b && c` through `addClauses` gives three sites, which is correct but unpinned.
No row has a keyed element in a package var value.
The `blockShadow` row shadows `text`, a variable with no literals, so a broken skip in `variableReads` changes no mutant.

### [LOW] A variable read only by another variable's value is never mutated

**Location:** internal/testcover/mutant.go:505

**Issue:** `variableReads` inspects `FuncDecl` bodies only.
For `var base = []string{"x"}` and `var all = append(base, "y")`, where only `all` is read in a function, `base` has no reader block and its literals are dropped.
This is a list-member gap of the kind the board entry describes, though none of the five named candidates depends on it.

### [LOW] A deleted clause still runs

**Location:** internal/testcover/mutant.go:66

**Issue:** `lymMutantDropAnd(advance(), id) && ready` still calls `advance()` when the mutant is on.
A real deletion would skip the call.
A test pinning only that side effect cannot kill the mutant.
None of the five candidates' clauses has a side effect, so the gap is narrow.
A wrapper that skips the evaluation would change typing for named bool operands, so the fix asks that the behaviour be stated rather than changed.

### [LOW] Block-less files are matched by the default build context

**Location:** internal/testcover/mutant.go:329

**Issue:** `BuildMutantBinary` passes no `-tags` of its own, but the holder environment can carry `GOFLAGS=-tags=...`.
`build.Default` does not read GOFLAGS.
A block-less file gated by such a tag can be skipped, and its excluded alternative can be scanned in its place.
A file with blocks always comes from the profile, so only declaration-only files are affected.

### [NIT] Object resolution is back on

**Location:** internal/testcover/mutant.go:345

**Issue:** The old call passed `parser.SkipObjectResolution`, and the new one passes `parser.ParseComments` alone.
Nothing reads `ast.Object`, and the summary says the change avoids the deprecated `ast.Object` resolution.

## Rejected

- Affected-set NIT on `pattern/PATTERN-test-speed.md:313`, which says "It mutates the candidates' blocks": every mutant now carries the blocks whose run reaches it, so the sentence still holds under the new attribution.
- Assistant's lexical-shadowing finding (`mutant.go:423`): the summary records whole-function shadowing as a deliberate trade-off toward fewer mutants without type information. The cases it misses, such as `prefix := prefix`, reach none of the five candidates, and the board entry does not ask for scope resolution.
- The assistant rated the untested helpers and the block attribution BLOCKING. They are merged into F1 and F2 at MEDIUM, since no repository rule makes a coverage gap blocking.
- Scope: every hunk traces to a goal point of the board entry or is forced by one, and every goal point has changes that build it (code-scope lens, no findings).

Ground truths I checked by hand against the diff:

- `holdFreeProducers` (`runlock.go:44`) is read in `RunHolds`, so its literals belong to that block.
  `TestRunHolds`'s `BeamAtWorktreeCreate` row kills the `"Worktree-Create"` mutant.
- `checkRunLocks:180` is an `||` chain.
  The candidate's `BeamChange` row changes only `Beam`, so deleting the Beam clause is a unique kill.
  Its `KindChange` row also changes Recipe and Beam, so deleting the Kind clause is not.
  One unique kill is enough.
- `endsAsSketch` (`sketch.go:81-84`) is an `&&` chain.
  Deleting `now.Status == nil` is killed only by `SketchThatKeepsItsRunStatusIsRefused`.
- `raddleStoreDirName` at `raddlestore.go:150` feeds the exact `git -C ... checkout -- raddle/broken/store.json` text, which `TestReadRaddleStore` asserts.
- The `template.yaml` lines behind `configTemplate` get one mutant per non-blank line, and the `design_prefix` and `labels` lines are asserted only by the candidate.
- Package-var initialisers call the helpers at init.
  Go's dependency order initialises `lymMutantActive` first, because the helpers' bodies read it.
- `BuildMutantBinary` passes `-gcflags=<pkg>=-e`, so the compiler's error cap does not exhaust the five build rounds on the new typed-context failures.

## Focus departures

- "Done-when: would `lym gate redundant internal/boardengine` stop listing the five named candidates": I did not run the verb.
  It needs a coverage map this worktree lacks, and building one is outside this review's write surface; the assistant's attempt was refused for that reason.
  I traced each candidate's unique kill through the source instead (see the ground truths above), and found no regression in the guard path of `internal/gatecli/redundantmutants.go`.
