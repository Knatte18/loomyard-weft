# Card 17 — loom-seed-driver-kind

**Edit:**
- `internal/loomcli#resolveSeedDriver` -> `func resolveSeedDriver(existing shedrun.Seed, found bool, kind string) string`
- `internal/loomcli#loomCLI.seedAndCommitBootstrap`
- `internal/loomcli#mustUseLLMDriverArm`
- `internal/loomcli#TestResolveSeedDriver`
- `internal/loomcli/doc.go`
- `pattern/PATTERN-driver-choice-single-site.md`

**Uses:**
- `plan:internal/loomengine#ResolveDriverKind`

**Intent:**
D1 at loom's own seed writer: a standalone `lyx loom start` on an unseeded run seeds the driver `driver_kind` resolves to, through the config `start` already loads strictly, instead of always `llm`.

`resolveSeedDriver` returns the recorded seed's driver when one exists, and `kind` otherwise.
`seedAndCommitBootstrap` passes `loomengine.ResolveDriverKind(c.cfg)`.
A seeded run's recorded driver never changes: an existing seed wins over the key, so editing `run_settings` or `loom.yaml` after seeding has no effect on that run.
The doc comments of `resolveSeedDriver`, `seedAndCommitBootstrap` (its "falling back to shedrun.DriverLLM" paragraph) and `mustUseLLMDriverArm` (its "An unseeded run is seeded with the llm driver" line) say the fallback is `driver_kind`, `llm` by default.
`doc.go`'s `# Start` section says an unseeded run takes `driver_kind`.

`PATTERN-driver-choice-single-site.md`'s permitted-consumer list says loom's seed writer reads `driver_kind` for an unseeded run, alongside the existing entry.
The key is read only at seeding, never by a producer, a generic verb or an engine.

Test `TestResolveSeedDriver` keeps its three cases and adds rows: unseeded with kind `go` returns `go`; unseeded with kind `llm` returns `llm`; seeded `llm` with kind `go` returns `llm`, and seeded `go` with kind `llm` returns `go`, so the recorded driver wins.

**ImpactSummary:** one pure resolver gains a parameter with its single caller in the same package; only an unseeded run's first `start` can differ, and only when `driver_kind` is `go`.
