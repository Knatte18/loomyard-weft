# Card 10 — shedrecipe-env-seats

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#shedrecipeAllowedImports`
- `internal/shedrecipe#newTestEnv`
- `internal/loomcli#loomCLI.wire`

**Uses:**
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/modelspec#Registry`
- `plan:internal/seatengine#New`
- `plan:internal/seatengine#RunnerShuttle`
- `plan:internal/seatengine#Geometry`
- `plan:internal/shedadapters#SeatRunner`
- `plan:internal/testkit/shedfake#SeatRunner`

**Intent:** Give the recipe `Env` the two run-wide values a `MultiLLM` row needs (D2, D12) and fill them in loom's wiring in the same card, since `TestWire_EverySeamFilled` reads every seam of `Env` through `envkit.NilSeams` and would fail between the two edits.
`Env` gains `Seats shedadapters.SeatRunner`, the seat runner over the caller's shuttle runner, nil when no row uses it, and `Models modelspec.Registry`, the registry a seat's `model` spec resolves against, with doc comments naming the entry that reads each; `shedrecipeAllowedImports` admits `internal/modelspec`, a leaf.
`loomCLI.wire` fills `Seats` with `seatengine.New(seatengine.RunnerShuttle(runner), seatengine.Geometry{...})` over the runner it already builds, the worktree path, the anchor path, webster's stencils directory, the parent name and the reed geometry's shortname and slug, and fills `Models` with the registry it already loads.
`newTestEnv` fills `Seats` with a `shedfake.SeatRunner` and `Models` with a small registry literal, so every later entry test has both.
No test is added: the wiring test above covers the fill, and the entry tests of card 11 cover the fields' use.

**ImpactSummary:** Two new Env fields; one wiring site and one test fixture fill them, and every other Env consumer is unchanged.
