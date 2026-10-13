# Card 18 — batten-child-driver

**Edit:**
- `internal/battencli#childDriverOf` -> `func childDriverOf(seed shedrun.Seed) (driver string, recorded bool)`
- `internal/battencli#refuseAdoptedSeed`
- `internal/battencli#battenCLI.armSeed`
- `internal/battencli#battenCLI`
- `internal/battencli#Command`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#TestRefuseAdoptedSeed`
- `internal/battencli#TestArmSeed_RunAndStepSeedBeforeWire`
- `internal/battencli#TestArmSeed_DriverFlagMatrix`
- `internal/battencli#TestWire_ChildDriverDelegatesToChildDriverOf`
- `internal/battenshed#seedChildProducer.Call`
- `internal/battenshed#SeedChildDeps`
- `internal/battenshed#NewSeedChild`
- `docs/overview.md`
- `internal/battenshed#TestSeedChild_RunSettings`
- `internal/battenshed#newSeedChildDeps`
- `pattern/PATTERN-driver-choice-single-site.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/battencli#battenChildDriver`

**Create:**
- `internal/battencli#childDriverFrom`
- `internal/battencli#TestChildDriverFrom`

**Uses:**
- `internal/loomengine#ResolveDriverKind`

**Intent:**
D1 at batten: a task's driver comes from an operator-typed `--child-driver`, then the task's resolved `driver_kind`, then `llm`.

`lyx batten run` and `lyx batten step` register `--child-driver` with an empty default, and its help says an unset flag takes the task's `driver_kind`.
`armSeed` records `child_driver` in the prime seed's params only when the flag was typed (`childDriverFlagSet`), after `shedrun.ValidateDriver`.
An untyped flag records none, and `battenChildDriver`, whose only caller was that write, is deleted.
The `battenCLI` field comments for `childDriverFlag` and `childDriverFlagSet` say so.

`childDriverOf` reports the recorded `child_driver` and whether the seed records one, so an absent param no longer means `go`.
`refuseAdoptedSeed`:
- a typed `--child-driver` against a found seed that records a different one is refused as today;
- a typed one against a found seed that records none is refused, naming the way forward: delete the seed at its path to re-seed it with the flag, or drop the flag to take `driver_kind`;
- an untyped flag passes a seed that records none.
This stays the one carve-out shape `PATTERN-driver-choice-single-site` allows: a typed value compared against a recorded one.

`childDriverFrom` returns the recorded `child_driver` when the prime seed records one, and otherwise `resolveKind()`; a `resolveKind` error is returned.
`wire`'s `SeedChild.ChildDriver` closure calls it with a `resolveKind` that locates the task worktree, loads its loom config strictly through `loomengine.LoadConfig(fabricengine.ConfigDirs(taskLocation), "loom")` (`PATTERN-config-strictness`), and returns `loomengine.ResolveDriverKind`.

`seedChildProducer.Call` calls `deps.ChildDriver` after `writeRunSettings`, so the strict load sees the local layer Seed-Child just wrote.
An invalid `driver_kind` in `run_settings` is refused earlier by `PrepareRunSettings` (card 2) and stands Stuck.
`SeedChildDeps.ChildDriver`'s comment, `NewSeedChild`'s doc comment and the `seamchild.go` file header say the driver is the prime seed's typed `child_driver`, else the task's `driver_kind`, no longer "inherited from prime's own seed".
`docs/overview.md`'s batten bullet replaces "`llm` by default" for `--child-driver` with: unset, the task's `driver_kind`, `llm` by default.
A seeded child's recorded driver never changes, since `WriteSeed` refuses a disagreeing child seed.

`PATTERN-driver-choice-single-site.md`'s permitted-consumer entry for batten's child-driver reader says it reads the typed param, else the task's `driver_kind`.
`refusal-spec.md` gains a row beside `Seed-Child: run settings invalid`: `batten run: typed --child-driver over a seed recording none`, a correctness halt with the way forward above.
The `## Out of scope` paragraph names that row beside the Seed-Child row.

Tests:
- `TestRefuseAdoptedSeed`: the row `an_absent_child_driver_param_compares_as_go` becomes a typed flag over a seed recording none, refused with both ways forward; it gains an untyped flag over a seed recording none, which passes.
- `TestArmSeed_RunAndStepSeedBeforeWire` asserts an unset child flag writes no `child_driver` param.
- `TestArmSeed_DriverFlagMatrix` marks its child flag typed on every row, so the written param and the unknown-value refusal stay pinned, and adds a row with the flag untyped that writes no param.
- `TestWire_ChildDriverDelegatesToChildDriverOf` keeps its explicit-`llm` row and drops the absent-param row, which now needs a task worktree.
- `TestChildDriverFrom` in `wire_test.go`, a table: a recorded param wins over `resolveKind`; no param, and no seed, each return `resolveKind`'s `go` and `llm`; a `resolveKind` error is returned.
- `TestSeedChild_RunSettings` gains a row asserting `ChildDriver` runs after every `WriteLocalOverride`, through a call-order log on `newSeedChildDeps`'s fakes.
`test-budget.yaml`'s `internal/battencli` row rises by one.

**ImpactSummary:** batten's prime seed records a child driver only when typed, Seed-Child resolves the rest from the task's `driver_kind` after writing the local layer, and one helper is deleted; every other prime-seed field and refusal is unchanged.
