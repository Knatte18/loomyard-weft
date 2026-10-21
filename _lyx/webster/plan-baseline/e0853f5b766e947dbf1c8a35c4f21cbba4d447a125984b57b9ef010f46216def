# Card 19 — non-tier-holders

**Create:**
- `pattern/PATTERN-gate-slot.md`
- `pattern/PATTERN-test-tiers.md`
- `pattern/PATTERN-recipe-setting-lookup.md`
- `cmd/testtiming#TestTakeGateSlot`
- `cmd/testtiming#withGateSlot`
- `internal/planglyph#goListLoader.loadEnv`

**Edit:**
- `internal/gateslot#Config`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#decodeConfig`
- `internal/gateslot#Config.Limits`
- `internal/gateslot#Limits`
- `internal/gateslot#Lease`
- `internal/gateslot#Pool.tryAcquireAny`
- `internal/gateslot#TestLeaseEnv`
- `internal/gateslot#TestLoadConfig`
- `internal/gateslot#TestTemplateConfigAndLimits`
- `internal/gateslot#singleSlotLimits`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#fixedLimits`
- `internal/gateslot#TestAcquire_ReadsLimitsAtEveryAcquire`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `internal/planglyph#goListLoader.load`
- `cmd/testtiming#takeGateSlot`
- `cmd/testtiming#run`
- `cmd/testtiming#runRedundancy`
- `cmd/testtiming#runResources`
- `cmd/testtiming#prebuildLyx`
- `cmd/testtiming#TestPrebuildLyx`
- `internal/hubgeom#TestWebsterGeometry`
- `internal/planglyph#TestGlyphChain_CallerCoverageTyped`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#TestConfigOverRealHub`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli#prebuildLyx`
- `cmd/testtiming#TestMain`
- `docs/overview.md`
- `docs/benchmarks/running-tests.md`
- `PATTERN.md`
- `pattern/PATTERN-test-speed.md`
- `pattern/PATTERN-push-both-sides.md`
- `CLAUDE.md`

**Delete:**
- `internal/gateslot#Lease.Env`
- `internal/gateslot#GoFlags`

**Uses:**
- `internal/gateslot#HolderEnv`

**Intent:**
Finish D13 and D1's retirement of `go_parallel`: every slot holder that is not a tier run sets its own Go flags, and the overview records the two declared PATTERN changes.

`go_parallel` becomes a retired key of the hub and machine layers (`CategoryMachine`, so an unreconciled machine file holding it still loads), dropped at load and pruned by reconcile; `Config.GoParallel`, `Limits.GoParallel`, the lease's stored cap, `Lease.Env` and `GoFlags` go, and the template line with them.
A slot's environment is now `Lease.InheritEnv` for a tier run (card 3) and `gateslot.HolderEnv` for a non-tier holder (card 11): GOFLAGS `-trimpath`, plus `-p=<tier0's parallel>` when the hub declares `tier0` and no `-p` when it does not, so every gate build keeps `-trimpath` (`PATTERN-trimpath-safe-root`).
`internal/planglyph`'s type load takes `HolderEnv` with the pool's `Limits().Tier0Parallel`, read at its acquire, keeping its `GOFLAGS=-mod=readonly` ahead of the appended flags.
`cmd/testtiming` holds one slot for its whole run in every mode, its default timing mode, `-redundancy` and `-resources`, its `lyx` prebuild included, which inherits that slot instead of taking its own: every mode runs inside the new `withGateSlot(cwd, site, mode)`, which calls `takeGateSlot` once, hands the mode `HolderEnv`'s environment and the `-p` value it sets as the mode's parallelism, holds the slot until the mode returns, and outside every hub hands the CPU count and holds no slot; each mode appends that environment to every `go` command it runs.
`internal/planglyph`'s loader builds its environment in the new `goListLoader.loadEnv`, which reads the pool's `Limits().Tier0Parallel` and calls `HolderEnv`, so the load's flags are testable without observing `go list`.
`cmd/testtiming` keeps measuring outside the gate, since its modes measure what a tier run hides.
`gateslot`'s doc lists the three non-tier holders, the card `**Verify:**` runner, the plan gate's type load and `cmd/testtiming`, and states their flags.

`PATTERN.md` makes the two declared changes, each line under the entry cap, with the declared text in full in the new background files:

```markdown
- `PATTERN-gate-slot` — Running a gate build or test: only in held `internal/gateslot` slots. — [background](pattern/PATTERN-gate-slot.md)
- `PATTERN-test-tiers` — Running a hub's tests: only through `lyx gate test`'s tiers. — [background](pattern/PATTERN-test-tiers.md)
```

`pattern/PATTERN-gate-slot.md` states the declared rule: only holding `internal/gateslot` slots, either inside a `lyx gate test` tier run, holding the slots its tier asks for, with the tier's config setting its parallelism and environment, or in one of the non-tier holders `internal/gateslot`'s doc lists, holding one slot and setting its own `-trimpath` Go flag, plus `-p` from `tier0`'s parallel when the hub declares `tier0`.
`pattern/PATTERN-test-tiers.md` states: only `lyx gate test` over `gate.yaml`'s tiers, apart from a card `**Verify:**` line and `cmd/testtiming`'s runs in every mode; a site or stencil names a gate's tier list and scope form, never a test command or target list; and it names the guards, the shell guard's `refuse_raw` row and the plan gate's `verify-raw-command`.
The overview may not grow past its headroom line, which it already sits above, so the card moves detail of two entries into background files to pay for the new line, keeping each rule whole there:

```markdown
- `PATTERN-recipe-setting-lookup` — Giving a recipe row a value: never its own `Env` field. — [background](pattern/PATTERN-recipe-setting-lookup.md)
- `PATTERN-push-both-sides` — Pushing run records from Go: the code branch goes along. — [background](pattern/PATTERN-push-both-sides.md)
```

`pattern/PATTERN-recipe-setting-lookup.md` holds that entry's current text whole, and `pattern/PATTERN-push-both-sides.md` gains the moved sentence that a per-transition status push goes through fabric's detached push; `lyx loom lint-pattern` passes with the overview no larger than at the base.
`pattern/PATTERN-test-speed.md`'s tier table and gate-per-tier section move to the hub's tier numbers, `tier0` untagged through `tier3` `llm`, and to D8's gates and their lists, and its slot section names tier runs and the non-tier holders.
`docs/benchmarks/running-tests.md`, a how-to guide rather than a measurement report, moves its tier section to the same numbers, `tier0` untagged through `tier3` `llm`, and its gate lines name `lyx gate test --tier <n>` in place of `--tags`.
`CLAUDE.md`'s push rule becomes `lyx gate test --tier 0 --tier 1 --tier 2` on `main`, where no parent makes the scope whole, and its gate wording drops `--tags`.

Forced edits: `TestGlyphChain_CallerCoverageTyped` builds its `Limits` with `Tier0Parallel` in place of `GoParallel`; gateslot's own `Limits` literals, `singleSlotLimits` in `testmain_test.go`, `fixedLimits` and `TestAcquire_ReadsLimitsAtEveryAcquire`, drop `GoParallel`, setting `Tier0Parallel` only where a test reads it, and the package-level `templateDefaults` `Config` literal drops `GoParallel`; `TestModules_EveryTemplateKeyClassified` drops `go_parallel` from the gate module's machine keys; `TestDispatchLayers` and `TestConfigOverRealHub` set `tier_parallel.tier0` on the machine layer in place of `go_parallel`; `TestGateTest_Scenario`'s machine `go_parallel` refusal row becomes the machine `tier_parallel` refusal; `prebuildLyx`'s doc stops naming `gateslot.GoFlags`; `docs/overview.md` names `slots` and `tier_parallel` as the gate's machine keys; `cmd/testtiming`'s `TestMain` calls `gitkit.HermeticGitEnv()`, and `TestTakeGateSlot` sits in an `integration`-tagged file, since its `hubforge` hub spawns git.
`hubreconcile`'s stale-build test seeds `go_parallel` in a machine file it only checks is left byte for byte, which a retired machine key keeps true, so it needs no edit.
Tests: `TestLeaseEnv` drops the `Env` rows; `TestLoadConfig` gains `go_parallel` loading as retired from the hub and the machine file, and `TestTemplateConfigAndLimits` drops `GoParallel`.
The new `TestTakeGateSlot` drives `withGateSlot` over a `hubforge` hub with a fake mode: one slot held under the `testtiming` site for as long as the mode runs and free after it returns, the mode handed GOFLAGS carrying the pool's `Limits().Tier0Parallel` through `HolderEnv` and that value as its parallelism, and outside every hub the CPU count with no slot; the flag rule itself stays pinned in `TestLeaseEnv` alone (card 11).
`TestPrebuildLyx` asserts the prebuild inherits the held slot and acquires none.
`TestGlyphChain_CallerCoverageTyped` gains a row asserting `loadEnv` keeps `-mod=readonly` and appends the pool's tier0 parallel through `HolderEnv`.
`TestWebsterGeometry` drops its lease GOFLAGS assertion; the machine override's mapping stays pinned in `TestTemplateConfigAndLimits` (card 11).
Its `Limits` literals take `Tier0Parallel` in place of `GoParallel`; the machine `go_parallel: 0` row, which asserts a load error, now loads because the key is retired, so it becomes a machine `tier_parallel: {tier0: 0}` row over a seeded `tier0`, refused naming both files; the machine override row seeds `tier_parallel.tier0`.

**ImpactSummary:** the last readers of `go_parallel` and the GOFLAGS-appending lease move to `HolderEnv`; three PATTERN background files are added and four overview lines change with no net growth.
