# Card 8 — wire-gate-findings-files

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#multiLLMEntry`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#prReworkEntry`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#describeEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe_test#TestMultiLLMFixtureRecipe_BuildsAndRunsAgainstFakeSeats`
- `internal/shedrecipe#TestDarnWriteEntry_HappyPath`
- `internal/shedrecipe#TestPlanWriteEntry_HappyPath`
- `internal/shedrecipe#TestDiscussionWriteEntry_HappyPath`
- `internal/shedrecipe#TestPRReworkEntry_HappyPath`
- `internal/shedrecipe#TestDescribeEntry_HappyPath`
- `internal/shedrecipe#TestBurlerFixEntry_GateConfig`
- `internal/shedrecipe#TestBurlerHalfEntries_RunDirectory`
- `internal/shedverbs#TestGotoCmd_MovesBlockedRunOntoRow`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`

**Uses:**
- `internal/shedrun#GateFindingsFile`

**Intent:** D4's wiring: every gated row in `internal/shedadapters` is told its own findings-file path, so the file its Stuck reason names exists after the shuttle run has finalized, Darn-Write's writer included.

`Env` gains `GateFindingsFile func(row string) string`, documented as the told per-row file an exhausted gate's findings are kept in; nil tells no row a path, which keeps the old reason.
Each entry that builds a `MultiLLMProducer` (`multiLLMEntry`, `planWriteEntry`, `discussionWriteEntry`, `prReworkEntry`, `darnWriteEntry`, `describeEntry`) chains `.KeepGateFindings(path)` onto it, the path `env.GateFindingsFile(row.Name)` when the seam is set, else empty; `burlerReviewEntry` and `burlerFixEntry` set `BurlerHalfDeps.GateFindingsPath` the same way.
A small unexported helper beside `Env` returns that path, so the nil check lives once.
`loomCLI.wire` sets `GateFindingsFile` to `func(row string) string { return shedrun.GateFindingsFile(location, c.runID, row) }`, so the geometry stays in the CLI wiring (`PATTERN-told-geometry`); `FullEnv` sets it to a file under its temp dir.
Batten's wiring has no gated row that needs the file and leaves the seam nil, so `internal/battencli#intentionallyNil` gains `"GateFindingsFile"` with that reason, keeping batten's `TestWire_EverySeamFilled` green.

Tests, each extending an existing test of the entry it pins, since card 7 pins the producer's write, name and removal:
- `TestMultiLLMFixtureRecipe_BuildsAndRunsAgainstFakeSeats` gains a run whose `shedfake.SeatRunner` returns a Done chair carrying a failed, non-terminal `GateOutcome` with `Findings: "the gate's findings"`: the Call is Stuck, its reason names `env.GateFindingsFile("Seat-Step")`, and that file holds the findings.
  A second run over an `Env` whose `GateFindingsFile` is nil asserts the reason names the outcome's `FindingsPath`, reaching the helper's nil branch.
- `TestDarnWriteEntry_HappyPath`, `TestPlanWriteEntry_HappyPath`, `TestDiscussionWriteEntry_HappyPath`, `TestPRReworkEntry_HappyPath` and `TestDescribeEntry_HappyPath` each script their seat runner's chair with that failed gate and assert the Call's reason names `env.GateFindingsFile` of the row's name, so an entry that drops the chain fails its own test, the Darn-Write writer's included.
- `TestBurlerFixEntry_GateConfig` gains a subtest whose fake burler runner returns a fix with that failed gate and asserts the reason names `env.GateFindingsFile("fix-row")`; `TestBurlerHalfEntries_RunDirectory` asserts a review Call whose gate passed removes a file pre-written at `env.GateFindingsFile` of the review row.
- `TestGotoCmd_MovesBlockedRunOntoRow` plants a file in the run's scratch directory, the directory of `paths.LockPath`, before the goto and asserts it is still there after, pinning that a goto leaves the findings file in place.
  A resume runs the bootstrap and the loop, which remove only the loop's pid files (`internal/shedverbs/loop.go:358`), and pinning that would need a live loop process, so no resume row is planned.
- `TestWire_DarnRecipe`'s darn subtest asserts `c.env.GateFindingsFile("Darn-Write")` equals `shedrun.GateFindingsFile(location, c.runID, "Darn-Write")`.

**ImpactSummary:** one optional `Env` seam read by the gated entries and set by loom's wiring; batten's seam list records its nil, and an Env built without it keeps today's reasons.
