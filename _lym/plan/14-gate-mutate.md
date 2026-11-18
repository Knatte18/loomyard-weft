# Card 14 — gate-mutate

**Create:**
- `plan:internal/gatecli#mutateRequest` -> `type mutateRequest struct`
- `plan:internal/gatecli#mutateCmd` -> `func mutateCmd(goBinary string, minuteUnit, grace time.Duration) *cobra.Command`
- `plan:internal/gatecli#runMutate` -> `func runMutate(ctx context.Context, out io.Writer, request mutateRequest) int`
- `plan:internal/gatecli#diffMutation` -> `type diffMutation struct`
- `plan:internal/gatecli#TestGateMutate` -> `func TestGateMutate(t *testing.T)`

**Edit:**
- `internal/gatecli#newCommand`
- `docs/overview.md`
- `pattern/PATTERN-test-speed.md`
- `internal/gatecli/doc.go`
- `internal/gateslot/template.yaml`

**Uses:**
- `plan:internal/testcover#PlanLineMutants`
- `plan:internal/testreach#ChangedLines`
- `plan:internal/testreach#Fresh`
- `plan:internal/testreach#ReadPackageMap`
- `plan:internal/testreach#Open`
- `plan:internal/testreach#Reader.TestsReaching`
- `plan:internal/gatecli#changedFuncsAt`
- `plan:internal/gatecli#reachStep`
- `internal/gatecli#mutationCheck.holdSlot`
- `internal/gatecli#mutationCheck.buildMutants`
- `internal/gatecli#mutationCheck.runJob`
- `internal/gatecli#redundantCmd`
- `internal/gatecli#TestGateRedundant`
- `internal/gatecli#seedTestMapModule`

**Intent:**
Add `lym gate mutate` (D11), which runs D3's refresh and then D9's mutation check over the worktree's diff, and file the trial's board note.
It joins the `gate` subtree in `newCommand` beside `redundant`, which it mirrors.

`lym gate mutate [--base <rev>]`:
- The base is the merge base with the parent branch, or `--base`; the changed functions and changed lines come from that base (through fabric) and the worktree's files (`changedFuncsAt`, `testreach.ChangedLines`).
- It runs card 6's refresh of the packages holding changed functions, under `reach_budget_min`; in the prime the refresh does nothing.
- Fresh maps (D9): a package's map is fresh when the refresh rewrote it, or its worktree anchor map, read through `testreach.ReadPackageMap`, passes `testreach.Fresh`;
  in the prime, which keeps no anchor maps, a shared map that passes `testreach.Fresh` is fresh.
  Every other changed package is listed as not judged with the reason and its way forward, and gets no mutant:
  uncommitted files, commit and run it again; past `reach_budget_min`, run it again; in the prime, a stale shared map, rebuild the maps with `lym gate test --map`.
- Per judged package, inside one gate slot (`PATTERN-gate-slot`) bounded by `mutation_budget_min` from the hold, `diffMutation` reuses `mutationCheck`'s machinery:
  it builds the coverage binary, reads the block table, plans with `testcover.PlanLineMutants` over the fresh map's covered blocks,
  then builds the mutant binary through the overlay (dropping unbuildable mutants as `buildMutants` does) — one binary per changed package.
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

`TestGateMutate` (tier1) runs the real `go` toolchain, as `TestGateRedundant`'s mutation rows do (`defaultGoBinary`), since its kill, survivor and unjudged rows build and run real mutant binaries,
over a seeded module (`seedTestMapModule`) and maps the test writes:
- a diff changing a covered condition yields a mutant the covering test kills, and the second covering test of that block is not run;
- a changed line no covering test checks yields a survivor with its file, line and operator, and a test of another package reaching it is listed as a possible killer;
- a changed uncovered block is listed uncovered with no mutant;
- a package whose anchor map is stale and whose files are committed is refreshed and then judged; a package with an uncommitted file is not judged, with its reason;
- a prime row with `--base` naming an earlier commit judges a package whose shared map matches the worktree;
- a cap of one runs one mutant; a cap of two over two changed functions plans one mutant in each;
- a mutant run past a timeout the row sets tiny is unjudged; a mutant run whose covering test allocates past a `redundancy_test_memory_mb` the row sets low is unjudged;
- `--base` moves the base so an earlier change is judged too;
- a hub whose scoper is not `go` gives its note with exit 0;
- one row per refusal.

Docs: `gatecli/doc.go` gains a `# Mutate verb` section, and its memory-guard sentence naming the runs the guard covers adds `lym gate mutate`;
`docs/overview.md`'s gatecli line names `lym gate mutate`; `pattern/PATTERN-test-speed.md`'s list of runs that take a tier's env and a gate slot names `lym gate mutate`;
`internal/gateslot/template.yaml`'s comments on `memory_quit_mb`, `redundancy_build_timeout_min`, `redundancy_test_timeout_sec` and `redundancy_test_memory_mb` add `lym gate mutate`.

The run files the trial's board note (D11) once this card's verb works, with `lym board upsert` and `kind: note` and one configured label, never as a task:
it asks for the trial the board entry describes, about twenty landed diffs replayed with `lym gate mutate`, counting survivors and reading a sample,
and asks the parent, from the trial's counts, whether to build D10's `mutation` report step and list `mutation` in the template's diff keys.

**ImpactSummary:** One new operator verb in the gate subtree reusing the redundant verb's mutant machinery and card 6's refresh; `newCommand` gains one line, and the overview, the test-speed background, the gatecli doc and the template comments each name the verb; one board note is filed.
