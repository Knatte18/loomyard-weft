# Card 20 — cleanup-verb

**Create:**
- `internal/cleanupcli/cli.go`
- `internal/cleanupcli/segments.go`
- `plan:internal/cleanupcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/cleanupcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/cleanupcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/cleanupcli#segment` -> `type segment struct`
- `plan:internal/cleanupcli#segmentResult` -> `type segmentResult struct`
- `plan:internal/cleanupcli#runSegments` -> `func runSegments(segments []segment, apply bool) ([]segmentResult, bool)`
- `plan:internal/cleanupcli#primeSegments` -> `func primeSegments(location *lyxcwd.Location) []segment`
- `internal/cleanupcli/cli_test.go`
- `internal/cleanupcli/cleanup_integration_test.go`

**Edit:**
- `internal/cleanupcli/doc.go`
- `cmd/lyx#newRoot`
- `cmd/lyx/seamsignature_test.go`
- `pattern/PATTERN-cli-cobra.md`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/battencli#Cleanup`
- `plan:internal/orchengine#CleanupHandoffs`
- `plan:internal/cleanupcli#scratchSegment`
- `internal/logger#SweepDir`
- `internal/logger#LogsDirAt`
- `internal/loggerconfig#LoadOrDefault`
- `internal/lyxtemp#ProcessRoot`
- `internal/lyxtemp#Root.Sweep`
- `internal/battencli#RefuseUnlessPrime`
- `internal/orchcli#PrimePaths`
- `internal/boardengine#OpenHub`
- `internal/boardengine#Board.Prune`
- `internal/boardengine#Board.ListTasksFull`

**Intent:**
Add `lyx cleanup [--apply]`, a top-level cobra module under the root in `cmd/lyx/main.go` that only calls each owning module's cleanup function in-process, so no module learns another's state (D7).
Without `--apply` it is a dry run that lists, per segment, what `--apply` would remove, with `"applied": false`; every segment computes its candidates with the same function in both modes.
It runs from the hub's prime worktree only: after resolving the cwd through `lyxcwd` it applies `battencli.RefuseUnlessPrime`, whose refusal says the verb runs from the prime worktree and to re-run it there, before any segment runs.
No `refusal-spec.md` row is added: batten and the cleanup module are not bound by `PATTERN-refusal-way-forward`.

`primeSegments(location)` returns the segments in this order, each a closure over its owner's function:

1. `batten`: `battencli.Cleanup(location, apply)`.
2. `board`: `lyx board prune`'s behavior, the hub board from `boardengine.OpenHub(location.HubPath)`, then `Board.Prune(apply)`.
3. `orch`: `orchengine.CleanupHandoffs(orchcli.PrimePaths(location), apply)`, which removes nothing outside the idle phase.
4. `scratch`: `scratchSegment` over `<prime worktree>/.scratch` with the same `OpenHub` board's `ListTasksFull` as its read and `time.Now()`.
5. `logs`: for the prime's anchor and `_board`'s directory, `logger.SweepDir(logger.LogsDirAt(anchor), loggerconfig.LoadOrDefault(anchor), apply)`, the bounds the exit sweep picks.
6. `temp`: `lyxtemp.ProcessRoot()` and its `Sweep(apply)`; the segment fails when the root fails its checks or the sweep reports a failure, and lists both its candidates and its failures.

`runSegments` runs every segment in order and never stops early: a failing segment's error goes into its own `segmentResult` and the next segment still runs; it reports ok false when any failed.
`Command` builds the invocable `cleanup` command with a lower-case `Short`, audience operator, `cobra.NoArgs` and `--apply`, its `RunE` checking `clihelp.ShouldAbort` first; it prints `{"applied", "segments"}` through `output.Ok` when every segment succeeded, and through `output.ErrFields` naming the failed segments otherwise, so the exit is non-zero.
`RunCLI` and `RunCLIIn` are the module's seams, as every module carries.
Bound: `lyx cleanup` removes nothing a segment's own verb would refuse to remove, since every refusal and skip lives in the module function both call and this verb adds only flag parsing, the prime check and output; it adds no `--force`, touches no remote except the records and board pushes batten cleanup and board prune already make, and no landing or batten row calls it.
It does not call `lyx fabric prune` or `lyx fabric cleanup` (D8).

Tests:
`cli_test.go`, untagged, covers `runSegments` with fake segments (order kept, a failing segment reported while the later ones still run, ok false), and `primeSegments` over a `locationkit` location under `t.TempDir()`: the segment names in order, and the `orch` segment over a seeded non-idle state with old handoffs removing nothing under `--apply`, as `lyx orch cleanup` does.
`cleanup_integration_test.go` (`//go:build integration`) builds a hub with `hubforge` and a task pair, seeds the prime with a done batten run directory, an old `.scratch/` child, old handoffs beside the one `LastHandoff` names and a done board task, writes the prime's `logger.yaml` with a short `trace_retention_days` bound and seeds one trace group in the prime's logs directory older than that bound and younger than the default, then asserts: a dry run lists every segment, that trace group under the `logs` segment, and changes nothing; `--apply` removes each seeded leftover, that trace group included; and the verb run in the task pair refuses naming the prime worktree.

Wiring and docs in this commit: `cmd/lyx`'s `newRoot` adds `cleanupcli.Command()` and its `Long` names `cleanup` among the available modules;
`cmd/lyx/seamsignature_test.go` pins `cleanupcli.RunCLI` and `cleanupcli.RunCLIIn`, and its header stops pinning a count of seams;
`pattern/PATTERN-cli-cobra.md`'s package-naming deviations add that `cleanupcli` calls the cleanup functions of `battencli`, `boardengine`, `orchengine`, `logger` and `lyxtemp`, with no engine package of its own;
`internal/cleanupcli/doc.go` describes the verb, the segment order, the prime-only rule and the bound;
`docs/overview.md` gains an `internal/cleanupcli/` row in the package tree and a **cleanup** bullet in the module list;
`tools/sandbox/SANDBOX-CORE-SUITE.md` gains a scenario tagged `**Covers:** cleanup` that runs `lyx cleanup` as a dry run in the sandbox's prime and watches for one `{"ok":true}` envelope with `"applied": false` and every segment listed, and for nothing removed (`PATTERN-sandbox-coverage`);
`cmd/lyx/testdata/test-budget.yaml`'s `internal/cleanupcli` row grows to the package's top-level test count.

**ImpactSummary:** A new root module wired into newRoot and the seam pins; it calls existing module functions and changes none of them.
