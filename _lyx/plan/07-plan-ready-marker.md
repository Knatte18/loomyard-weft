# Card 7 — plan-ready-marker

**Create:**
- `plan:internal/loomengine#PlanReadyMarker` -> `func PlanReadyMarker(l *lyxcwd.Location) string`
- `plan:internal/loomengine#RemovePlanReadyMarker` -> `func RemovePlanReadyMarker(l *lyxcwd.Location) error`
- `plan:internal/shedrecipe#TestPlanWriteEntry_PreparePlanSpawn` -> `func TestPlanWriteEntry_PreparePlanSpawn(t *testing.T)`

**Edit:**
- `internal/loomengine#TestDiscussionPaths`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe#TestPlanWriteEntry_ConstructionFailures`
- `internal/shedrecipe#newTestEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/testkit/envkit#FullEnv`
- `internal/battencli#intentionallyNil`

**Uses:**
- `internal/loomengine#DiscussionReadyMarker`
- `internal/loomengine#RemoveDiscussionReadyMarker`
- `internal/planparser#PlanDirName`
- `internal/planparser#PlanOverview`

**Intent:**
D6: the plan ready marker, and its removal before every fresh Plan-Write spawn.

`internal/loomengine`, beside `DiscussionReadyMarker` and `RemoveDiscussionReadyMarker` in `config.go`:
- `PlanReadyMarker(l)` returns `<anchor>/<lyxdirs.DotLyxDirName>/<planparser.PlanDirName>/00-overview.md.ready`, the overview's base name taken from `planparser.PlanOverview`, so neither segment is spelled a second time (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`: untracked state under `.lyx` at the subpath mirroring the plan directory).
  Its doc says it is the twill plan writer's ready marker, created once its plan passes the self-check, and the condition its forks wait on.
- `RemovePlanReadyMarker(l)` removes it and treats a missing marker as success, wrapping any other error as `loom: remove plan ready marker: %w`.
It is not an output file: `PlanSpec`'s `OutputFiles` and gates stay unchanged.

`internal/shedrecipe`:
- `Env` gains `PreparePlanSpawn func() error`, documented like `PrepareDiscussionSpawn`: the closure every fresh Plan-Write spawn runs after the attach probe and before the plan-directory rotation, arriving as a closure for the reason `DiscussionSpec` gives.
- `planWriteEntry` requires it with `requireSeam("PlanWrite", "PreparePlanSpawn", env.PreparePlanSpawn)` and hands the producer a preparation that runs `env.PreparePlanSpawn()` first, returning its error before anything else, and only then the rotator `loomshed.NewPlanDirRotator` builds, whose amendment it returns.
  The removal is its own step, ahead of the rotation, because the rotation returns early when the plan directory holds no stale file.
  Attaching to a live writer or relaunching a dead one runs no preparation (`SingleLLMProducer` prepares only after its attach probe finds nothing), so the marker is kept there.
  Its doc comment names the new seam.

Wiring and fixtures:
- `loomCLI.wire` fills `PreparePlanSpawn: func() error { return loomengine.RemovePlanReadyMarker(location) }` beside `PrepareDiscussionSpawn`, with a comment that it removes a marker an earlier writer spawn left whichever producer is configured.
- `envkit.FullEnv` and `shedrecipe`'s `newTestEnv` fill it with a nil-returning closure; `battencli`'s `intentionallyNil` lists `"PreparePlanSpawn": "loom-only seam, batten has no plan segment"`, so both `TestWire_EverySeamFilled` tests keep passing.

Tests:
- `TestDiscussionPaths` also pins `PlanReadyMarker(l)` as `filepath.Join(l.AnchorPath(), lyxdirs.DotLyxDirName, planparser.PlanDirName, "00-overview.md.ready")` at both rows; its doc line names the plan marker.
- `TestPlanWriteEntry_ConstructionFailures` gains a `NilPreparePlanSpawn` subtest asserting the error names `PlanWrite` and `PreparePlanSpawn`.
- New `TestPlanWriteEntry_PreparePlanSpawn`, a table over a `newTestEnv` with a counting `PreparePlanSpawn` that also records whether the plan directory's stale `00-overview.md` still existed when it ran:
  - fresh spawn with no plan directory: one call, the no-stale-files path (acceptance: "including one whose plan directory holds no stale files");
  - fresh spawn over a stale plan, with the prior-plan stencil seeded as `TestPlanWriteEntry_CallAppendsPriorPlanBlock` does: one call, made while the stale file was still in place, so it ran before the rotation;
  - attach to a live writer (`AttachFound`, `AttachResult` Done): no call;
  - relaunch of a dead writer (`AttachFound`, `AttachResult` Died): no call;
  - a `PreparePlanSpawn` returning an error: `Call` returns an error carrying it, no spec reaches the fake shuttle and the stale plan file is not rotated.
- `TestWire_DefaultConfig`'s "plan spec evaluates to the expected shape" subtest plants a file at `loomengine.PlanReadyMarker(loc)` and asserts `c.env.PreparePlanSpawn()` removes it, returns nil again with the marker absent, and returns an error when a regular file sits where the marker's directory belongs, as `TestWire_DiscussionSpecEvaluatesToExpectedShape` does for the discussion marker.
`internal/shedrecipe` stays within its test budget row.

**ImpactSummary:** `shedrecipe.Env` gains a required seam that `planWriteEntry` refuses nil; every builder of an `Env` (loom's wire, the envkit and shedrecipe fixtures) fills it and batten lists it as intentionally nil.

**Commit:** `7: Remove a stale plan ready marker before every fresh Plan-Write spawn`
