# Coverage maps: reach note, failure triage and `lym gate mutate`

The per-test coverage maps of `internal/testcover` now serve three uses over one reader:
a reach note for a diff, triage lines in failure reports, and a mutation check of the lines a diff changes.
Test selection and ordering from the maps stay retired, and `lym gate test --map` measures a package as before.

## What changes

- **`internal/testreach` (new)** is the one cross-package reader of the maps.
  It answers which tests reach a function and which functions of a set a test reaches, and it finds a diff's changed functions by function text.
  A function moved verbatim between files of its package is unchanged, and a deleted function is not a changed function.
  A reverse index is cached beside the shared maps.
  Only `lym gate test --map` run from the prime writes it; a reader with an absent or stale index builds the table in memory.
  A worktree map whose file hashes match the worktree wins over the hub's map for its package.
- **`reach` report step** of `lym gate test`.
  After the tiers pass it refreshes the worktree's own maps of the packages holding changed functions, inside a gate slot, bounded by `reach_budget_min`.
  It writes `reach.json` and prints a note to stderr naming, per changed function, up to five reaching tests (own package first, fewest-reach first) and the map commits read.
  It lists the unreached functions, with caveats for tier 2 tests, spawn-marked entries and new functions that older maps of linked packages may reach.
  The step is advisory: it never changes the gate's exit.
  A package with uncommitted files (its own or a linked package's), one past the budget and every package of a call in the prime are answered from the maps there are, with a line saying why.
- **Reach turn** for the loomshed verify gate and the weaver batch gate.
  After a passing verify, a matching `reach.json` that lists an unreached function no earlier turn listed returns one failing result carrying the note, so the writer adds tests itself.
  Each function listed once, with its text hash, never brings another turn, and the turn is skipped when one more failure would exhaust the entry's attempts.
- **`internal/gatefixer`**: `GateResult.WriterOnly` keeps a reach-turn result from spawning a cold fixer at Darn-Write, and `RerunMark` marks the wrap's post-fixer re-run so the attempt count is one per failing arrival.
- **Code review rubric** gains a `reach_report` marker, used only when the report matches HEAD's tree (changed functions, their hashes, and the Go files of changed and named packages).
  The rubric reads the note as evidence for "a behavior the diff adds with no coverage"; an unreached function is a candidate, not a finding.
- **`lym gate reach <file>#<Func>`** prints the note for one function from stored maps with no build and writes nothing.
  **`lym gate reach --test <pkg>.<Test> --since <rev>`** walks the parent branch's first-parent commits (cap 200, `--branch` to pick another) and lists those whose changed functions the test reaches.
  Each refusal names its way forward.
- **Failure triage**: each failed test a map holds gets a first output line in the failure report naming the run's changed functions it reaches, or "reaches none of this run's changes" followed by the `--since` command on its own line.
  With no parent branch (including in the prime) or an unreadable merge base, the line says no change set exists and why.
  Triage lines are dropped before any test output is cut, and `gateslot.FailedTests` parses reports unchanged.
  Every fixer's report carries the lines of its failed tests.
- **`lym gate mutate`** runs the refresh and then a changed-line mutant planner added to `internal/testcover`.
  Mutants land only on changed lines in covered blocks, one per line, in operator order: negated condition, deleted `&&`/`||` operand, dropped statement, shifted integer constant.
  String constants, embedded file lines and logger calls get none.
  `mutation_max_mutants` caps the total, and each mutant runs only the tests covering its block until one fails.
  Survivors, uncovered blocks and packages not judged are printed; `--base <rev>` picks another base.
- **`gate.yaml` template**: `reach` is listed first among the reports of every key that lists `redundancy` and in `batch`.
  New keys `reach_budget_min`, `mutation_max_mutants` and `mutation_budget_min` get defaults, and existing hubs pick them up through config load.
- `PATTERN-coverage-map-reader` (new) pins the reader as the only cross-package map reader, with a guard test in `cmd/lym`.
  `PATTERN-test-speed` and `PATTERN-gate-slot` are updated for the worktree map refresh.

## Design decisions

- The `mutation` report step is designed in the decision record but not built.
  It waits on the trial: about twenty landed diffs replayed with `lym gate mutate`, with survivors counted and a sample read.
  The config check accepts `reach` but not `mutation`.
- The reach refresh always writes the worktree's anchor maps, never the hub's shared maps, which stay written from the prime alone.
- All caps (note lines, tests per function, triage functions, the 200-commit walk) are named constants, not config keys.
- Only Go is scoped; any other scoper gets a note naming it, as the redundancy step does.

## To check by hand

- Tier 2 and tier 3 tests were not run.
- Run `lym gate mutate` on a real diff, and `lym gate reach <file>#<Func>` on a function, to see the output form on a live hub.
- Triage in the prime (no parent branch, so the no-change-set line and no D8 command) has no test asserting it.
- The wiring of the reach seam into the gate has no assertion that the real path is connected.
- The reach turn's record of functions already listed (`internal/verifytree/reach.go`) should be checked for forgetting a listed function across arrivals.
- The trial's board note asks the parent whether to build the `mutation` step and list it in the template's diff keys; confirm it was filed.
