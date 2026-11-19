# Card 13 — gate-mutate

**Create:**
- `internal/testreach#ChangedLines`
- `internal/testreach_test#TestChangedLines`
- `internal/testcover#PlanLineMutants`
- `internal/testcover_test#TestPlanLineMutants`
- `internal/testcover_test#lineMutantFixture`
- `internal/gatecli#mutateRequest`
- `internal/gatecli#mutateCmd`
- `internal/gatecli#runMutate`
- `internal/gatecli#diffMutation`
- `internal/gatecli#TestGateMutate`

**Edit:**
- `internal/testcover#site`
- `internal/testcover#fileSites`
- `internal/gatecli#newCommand`
- `docs/overview.md`
- `pattern/PATTERN-test-speed.md`
- `internal/gatecli/doc.go`
- `internal/gateslot/doc.go`
- `internal/gateslot/template.yaml`

**Extends:**
- `internal/testreach/doc.go`
- `internal/testcover/doc.go`

**Uses:**
- `internal/testcover#PlanMutants`
- `internal/testcover#MutantPlan`
- `internal/testcover#Mutant`
- `internal/testcover#KeyBlocks`
- `internal/testcover_test#planFixture`
- `internal/testcover_test#TestPlanMutants`
- `internal/testreach#Fresh`
- `internal/testreach#ReadPackageMap`
- `internal/testreach#Open`
- `internal/testreach#Reader.TestsReaching`
- `internal/gatecli#changedFuncsAt`
- `internal/gatecli#reachStep`
- `internal/gatecli#mutationCheck.holdSlot`
- `internal/gatecli#mutationCheck.buildMutants`
- `internal/gatecli#mutationCheck.runJob`
- `internal/gatecli#redundantCmd`
- `internal/gatecli#TestGateRedundant`
- `internal/gatecli#seedTestMapModule`

**Intent:**
Add D9's changed-line mutant planner and `lym gate mutate` (D11), which runs D3's refresh and then the mutation check over the worktree's diff, and file the trial's board note.
The line diff and the planner land with the verb, their first caller.

`testreach.ChangedLines(base, head)` returns the sorted 1-based line numbers of head that a line diff of the two texts marks added or changed.
An empty base marks every line of head.

`testcover.PlanLineMutants(dir, importPath, profile, changed, covered)` places mutants only on the lines a diff adds or changes, in blocks a test covers:
- `profile` is the package's block table from its coverage binary (`Toolchain.ProfileBlocks`), keyed with `KeyBlocks` as `PlanMutants` keys it;
  `changed` maps a file name of the package to its changed line numbers (`testreach.ChangedLines`, which the verb computes);
  `covered` holds the rendered keys of the blocks a fresh map's measured tests cover.
- A block holding a changed line that no key of `covered` names gets no mutant; it is returned in the second result, as an uncovered changed block.
- Operators, in this order: a negated condition or comparison, a deleted operand of `&&` or `||`, a dropped statement and a shifted integer constant (or rune).
  A string constant, an embedded file line and a dropped call into `internal/logger` get no mutant; the logger's import path is one named constant.
- At most one mutant per changed line, the first site the operator order gives; a site counts for the line its start lies on.
- The plan is a `MutantPlan` like `PlanMutants`', so `WriteOverlay`, `BuildMutantBinary`, `UnbuildableMutants` and `MutantEnv` serve it unchanged.
  The cap of D9 (`mutation_max_mutants`) is the caller's, across packages; each mutant's `Blocks` names its block, whose `Func` lets the caller spread the cap over the changed functions.

`site` gains `callee`, the import path of the callee's package for a dropped call statement, read from the file's imports; it is empty for a local or method call and for every other operator.
`fileSites` keeps its signature and fills it.
`PlanLineMutants` reads `callee` to pass over a call into `internal/logger`; `PlanMutants` ignores it, so its sites and order stay as they are.

`lym gate mutate [--base <rev>]` joins the `gate` subtree in `newCommand` beside `redundant`, which it mirrors:
- The base is the merge base with the parent branch, or `--base`; the changed functions and changed lines come from that base (through fabric) and the worktree's files (`changedFuncsAt`, `testreach.ChangedLines`).
- It runs card 5's refresh of the packages holding changed functions, under `reach_budget_min`; in the prime the refresh does nothing.
- Fresh maps (D9): a package's map is fresh when the refresh rewrote it, or its worktree anchor map, read through `testreach.ReadPackageMap`, passes `testreach.Fresh`;
  in the prime, which keeps no anchor maps, a shared map that passes `testreach.Fresh` is fresh.
  Every other changed package is listed as not judged with the reason and its way forward, and gets no mutant:
  uncommitted files, commit and run it again; past `reach_budget_min`, run it again; in the prime, a stale shared map, rebuild the maps with `lym gate test --map`.
- Per judged package, inside one gate slot (`PATTERN-gate-slot`) bounded by `mutation_budget_min` from the hold, `diffMutation` reuses `mutationCheck`'s machinery:
  it builds the coverage binary, reads the block table, plans with `testcover.PlanLineMutants` over the fresh map's covered blocks,
  then builds the mutant binary through the overlay (dropping unbuildable mutants as `buildMutants` does) — one binary per changed package.
- Each judged package's coverage build and planning run whatever the budget, each build bounded by `redundancy_build_timeout_min`, so every judged package's mutants are planned.
  Past `mutation_budget_min` the check starts no further mutant build or run, and every mutant left is reported unjudged with the budget reason and counted in the unjudged count.
- `mutation_max_mutants` caps the mutants of the diff, taken one changed function at a time in turn until the cap or the plans run out.
- Each mutant runs the fresh map's tests that cover its block, alone, until one fails: the first failing test kills it, and the covering tests after it are not run.
  A run that ends at its time bound or the memory cap is unjudged, never a kill, as `lym gate redundant` treats it; a mutant every covering test passes is a survivor.
- A survivor whose function the reader shows reached by tests of other packages lists those tests as possible killers; they are not run.
- It prints an ok envelope: the survivors (file, line, operator, possible killers), the uncovered changed blocks, the packages not judged with reasons, and counts of mutants run, killed, survived and unjudged — the form D10's summary names.
  It exits 0 once it ran, whatever it found.
- A hub whose scoper is not `go` judges nothing: the envelope carries a note naming the scoper, as the report steps do, exit 0.
- Refusals with their way forward: a call outside a hub; a worktree with no parent branch and no `--base` (pass `--base`), the prime's case; a `--base` that names no commit (pass one that does).
  Each is a row of `TestGateMutate`.
- Bound: it is a hand verb for the trial; it changes nothing but the worktree's anchor maps its refresh writes, and the budgets bound its time.

Tests:
- `TestChangedLines` (tier0) is one table: an inserted line, a changed line, a deletion (no head line marked), and an empty base.
- `lineMutantFixture` writes a package of its own into `t.TempDir()` and returns its directory, leaving `planFixture` and the constants `TestPlanMutants` pins unchanged:
  a file importing `internal/logger` with a call into it, a condition and an integer constant on one line, a string constant, an embedded file and its use.
  `fileSites` parses without type checking, so the import path is recognised in a temp directory.
- `TestPlanLineMutants` (tier0) is one table over `lineMutantFixture`'s files:
  mutants land only on changed lines of covered blocks, at most one per line, the first by operator order (a line holding a condition and a constant gets the negation);
  a changed line holding only a string constant, a use of an embedded file and a `logger` call gets none;
  a changed line in an uncovered block gets no mutant and its block comes back uncovered; an unchanged covered line gets none;
  the plan's overlay builds through `WriteOverlay` as `TestMutantPlan_WriteOverlay` does.
- `TestGateMutate` (tier1) runs the real `go` toolchain, as `TestGateRedundant`'s mutation rows do (`defaultGoBinary`), since its kill, survivor and unjudged rows build and run real mutant binaries,
  over a seeded module (`seedTestMapModule`) and maps the test writes:
  - a diff changing a covered condition yields a mutant the covering test kills, and the second covering test of that block is not run;
  - a changed line no covering test checks yields a survivor with its file, line and operator, and a test of another package reaching it is listed as a possible killer;
  - a changed uncovered block is listed uncovered with no mutant;
  - a package whose anchor map is stale and whose files are committed is refreshed and then judged; a package with an uncommitted file is not judged, with its reason;
  - a prime row with `--base` naming an earlier commit judges a package whose shared map matches the worktree;
  - a cap of one runs one mutant; a cap of two over two changed functions plans one mutant in each;
  - a mutant run past a timeout the row sets tiny is unjudged; a mutant run whose covering test allocates past a `redundancy_test_memory_mb` the row sets low is unjudged;
  - two budget rows each drive `mutateCmd` with a `minuteUnit` so short that the budget is spent at the hold:
    under `reach_budget_min`, a package with a stale anchor map is listed not judged with its budget line;
    under `mutation_budget_min`, the planned mutants of a package whose anchor map matches are reported unjudged with the budget reason and counted unjudged.
    The coverage build's bound and the budget share the one `minuteUnit`, so only the config values separate them:
    the `mutation_budget_min` row passes a `minuteUnit` of one millisecond,
    and its hub config sets `mutation_budget_min: 1` and `redundancy_build_timeout_min: 100000`;
    the budget is then one millisecond and the build bound 100 seconds, far above the real coverage build time,
    so the build finishes and the mutants are planned before the spent budget leaves them unjudged;
  - `--base` moves the base so an earlier change is judged too;
  - a hub whose scoper is not `go` gives its note with exit 0;
  - one row per refusal.

Docs:
- `testreach/doc.go` gains `ChangedLines`; `testcover/doc.go`'s mutation paragraph gains the changed-line planner and its operator set.
- `gatecli/doc.go` gains a `# Mutate verb` section, and its memory-guard sentence naming the runs the guard covers adds `lym gate mutate`.
- `internal/gateslot/doc.go`'s paragraph on the non-tier holders names `lym gate mutate` as a third one-slot holder:
  it waits for its slot after its refresh and holds it to its end.
  Its bounds paragraph adds `lym gate mutate` to the sentences naming the runs that take the three `redundancy_*` bounds and the runs that run fewer tests side by side.
- `docs/overview.md`'s gatecli line names `lym gate mutate`, and its sentences on the memory cap, the build and test bounds and the memory guard name it beside `lym gate redundant`.
- `pattern/PATTERN-test-speed.md`'s list of runs that take a tier's env and a gate slot names `lym gate mutate`.
- `internal/gateslot/template.yaml`'s comments on `memory_quit_mb`, `redundancy_build_timeout_min`, `redundancy_test_timeout_sec` and `redundancy_test_memory_mb` add `lym gate mutate`.

The run files the trial's board note (D11) once this card's verb works, with `lym board upsert` and `kind: note` and one configured label, never as a task:
it asks for the trial the board entry describes, about twenty landed diffs replayed with `lym gate mutate`, counting survivors and reading a sample,
and asks the parent, from the trial's counts, whether to build D10's `mutation` report step and list `mutation` in the template's diff keys.

**ImpactSummary:** One new operator verb in the gate subtree reusing the redundant verb's mutant machinery and card 5's refresh, with its line diff in `internal/testreach` and a changed-line planner beside `PlanMutants` sharing its site scan; `site` gains a callee field that `PlanMutants` ignores and `fileSites` keeps its signature, so its plans and tests are unchanged; `newCommand` gains one line, and the overview, the test-speed background, the gatecli and gateslot docs and the template comments each name the verb; one board note is filed.
