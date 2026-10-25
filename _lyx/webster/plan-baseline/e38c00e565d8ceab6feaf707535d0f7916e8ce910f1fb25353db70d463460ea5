# Card 14 — drift-recipe-rows

**Create:**
- `plan:internal/loomshed#NameDriftBaseline` -> `const NameDriftBaseline = "Drift-Baseline"`
- `plan:internal/loomshed#NameDriftAwait` -> `const NameDriftAwait = "Drift-Await"`
- `plan:internal/loomshed#NameDriftVerify` -> `const NameDriftVerify = "Drift-Verify"`
- `plan:internal/shedrecipe#driftBaselineEntry` -> `func driftBaselineEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#driftAwaitEntry` -> `func driftAwaitEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#driftVerifyEntry` -> `func driftVerifyEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestDriftEntries` -> `func TestDriftEntries(t *testing.T)`
- `plan:internal/loomcli#driftDeps` -> `func driftDeps(location *lyxcwd.Location, statusPath, statusLockPath string, awaitRow bool, tables driftTables) driftengine.Deps`
- `plan:internal/loomcli#driftTables` -> `type driftTables struct`
- `plan:internal/loomrecipe#TestDriftRows_EntryGate` -> `func TestDriftRows_EntryGate(t *testing.T)`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#deriveSequenceOrder`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestDarnRouting`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/reportengine/doc.go`
- `internal/loomshed#InterruptPolicies`
- `internal/reportengine#loomPhases`
- `internal/reportengine#darnPhases`
- `internal/loomrecipe/doc.go`
- `docs/overview.md`

**Extends:**
- `internal/shedrecipe/doc.go`

**Uses:**
- `plan:internal/driftengine#NewDriftBaseline`
- `plan:internal/driftengine#NewDriftAwait`
- `plan:internal/driftengine#NewDriftVerify`
- `plan:internal/driftengine#Deps`
- `plan:internal/driftengine#BaselineTable`
- `plan:internal/driftengine#VerifyTable`
- `internal/driftengine#StartCapture`
- `internal/driftengine#RunCapture`
- `internal/driftengine#RunCompare`
- `internal/driftengine#SkipList`
- `internal/driftengine#CheckMainCapture`
- `internal/shedrecipe#resolveRowGateSpec`
- `internal/loomcli#pairMergeBase`
- `internal/loomcli#commitDiscussion`
- `internal/loomcli#intentionallyNil`

**Intent:**
Wire the three rows into loom and darn (D2), register their engines (PATTERN-shed-recipe-registry) and fill their seams.

`loom-recipe.yaml`: `Discussion-Bouncer` → `Drift-Baseline` → `Plan-Write`; `Plan-Bouncer` → `Drift-Await` → `Webster`; `Code-Bouncer` → `Drift-Verify` → `Describe`.
`darn-recipe.yaml`: `Preflight` → `Drift-Baseline` → `Darn-Write` → `Drift-Verify` → `Code-Bouncer`, the `code_max_bounces: 0` comment moving from `Darn-Write`'s edge to `Drift-Verify`'s.
Every Drift row carries `entry_gate: drift.enabled` (the parent's ruling) and no `on_stuck`.
Engines and config: `Drift-Baseline` is `engine: DriftBaseline` with `subject: decision-record` in loom and `subject: board-entry` and `capture: wait` in darn; `Drift-Await` is `engine: DriftAwait` with no config; `Drift-Verify` is `engine: DriftVerify` with the row's `subject`, `gate_key: drift-verify` and `gates: [{name: verify, attempts: 3}]`.
Each header comment's main line, its no-`on_stuck` sets and its uncompared paths (darn's Code perch fixes, a Finalize merge and fixer, darn's Finalize reroute; a loom Finalize reroute re-enters `Drift-Verify` through `Parent-Align` and the Code perch) are updated.
What the entry gate lets through, and its bound: with `drift.enabled` false a run records each Drift row `Skipped` and spawns or runs nothing; the gate is evaluated only when the run enters the row, so a flag turned off at an entered row (waiting or halted) does not skip it (D2, D5).

`loomshed.go` gains `NameDriftBaseline`, `NameDriftAwait` and `NameDriftVerify` for the coverage guard, and `InterruptPolicies` maps all three to `shedverbs.InterruptPolicyReinvoke`, which `interruptpolicy_meta_test.go` requires of every assembled row: the two agent rows' seat producer probes for a live chair and reattaches, and `Drift-Await` spawns nothing.
`reportengine`'s `loomPhases` maps `Drift-Baseline` and `Drift-Await` to `PhasePlan` and `Drift-Verify` to `PhaseWebster`, beside `Code-Bouncer`; `darnPhases` maps `Drift-Baseline` to `PhaseSetup` and `Drift-Verify` to `PhaseReview`; so the timing report names a phase for every row.
`reportengine`'s `doc.go` phase prose follows: its loom sentence names `Drift-Baseline` and `Drift-Await` under plan and `Drift-Verify` under webster, and its darn sentence `Drift-Baseline` under setup and `Drift-Verify` under review.
`shedrecipe`'s `registry` gains `DriftBaseline`, `DriftAwait` and `DriftVerify`.
`Env` gains `Drift driftengine.Deps`, a whole-struct passthrough following `Env.Darn`, read by the three Drift entries only.
Each entry rejects unknown keys naming the way forward, validates the `Env.Drift` seams it reads through `requireSeam`, and fills `Drift.Seats` and `Drift.Now` from `Env.Seats` and `Env.Now` and, for `Drift-Verify`, `Drift.GateFindingsFile` from `Env.GateFindingsFile` for the row, as the `Command` entry fills its settings:
`driftBaselineEntry` requires `subject` to be `decision-record` or `board-entry` and admits `capture` `detach` (default) or `wait`;
`driftAwaitEntry` takes no key;
`driftVerifyEntry` requires `subject`, resolves `gates` through `resolveRowGateSpec` and requires exactly one entry, named `verify`, which it hands to `NewDriftVerify`.
`shedrecipe`'s `doc.go` gains a "The Drift rows" section; `loomrecipe`'s `doc.go` loom and darn tails name `Drift-Verify` after Code-Review in loom and between `Darn-Write` and Code-Review in darn, and `Drift-Baseline` and `Drift-Await` in their places; `docs/overview.md`'s darn recipe paragraph and the loom and darn run tails name the three rows and the `drift.enabled` switch, and its execution stack gains a bullet that regression sits beside the stack: the Drift rows are shed producers over seats, and the capture is a detached registered process, not a gate tier.

`loomCLI.wire` sets `c.env.Drift = driftDeps(...)` for loom and darn alike, `awaitRow` true for loom only.
`driftDeps` builds the seams over the worktree: `NewPaths(location.AnchorPath())`; `LoadConfig(fabricengine.ConfigDirs(location))`; the run status read under its lock; HEAD, the branch point through `pairMergeBase`, and the dirty paths as `fabricengine.SourceDirtyPaths(location)`, the code worktree's tracked dirt, never `Fabric.DirtyPaths`, which also lists the records checkout; `Commit` as `fabricengine.CommitAnchoredPaths` over the told paths, as `commitDiscussion` commits its directory; `BaselineTable` and `VerifyTable` over the stencils directory, registry, slug, the decision record path or board entry, the drift paths and the hub's language skills, which `driftTables` carries from `wire`; `StartCapture` over `os.Executable`, `fabricengine.RunProcDir(location.HubPath)` and the worktree name as owner; `RunCapture` and `Compare` over `hubgeom.GateSlots`, `gateslot.WaitDir` and the scope's `SkipList`, the capture's `Check` re-reading status, HEAD and dirt for `CheckMainCapture`, `RunCapture` running in the step's own process with no named job, so its command dies with the step; and a context-aware `Sleep`.
`envkit.FullEnv` fills `Drift` with temp paths and no-op seams, and `nilLegal` gains `Drift.Seats` and `Drift.Now`, which the entries fill, so `loomcli`'s `intentionallyNil` needs no entry; `battencli`'s `intentionallyNil` gains every other `Drift.*` seam as loom- and darn-only.

Tests:
`wantProducerTable` and `wantDarnProducerTable` gain the rows in recipe order with their engines and `driftengine` producer types; `deriveSequenceOrder` gains a branch giving each Drift row a `skipped` entry, since the kit's template answers `drift.enabled` false, so `wantSequenceOrder` follows; `TestNewDarn_Routing` and `TestDarnRouting` expect `Darn-Write` → `Drift-Verify` → `Code-Bouncer` and `Preflight` → `Drift-Baseline` → `Darn-Write`; `TestRouting_ProgressAt`'s tail subtest, whose name gains `Drift-Verify` after the Code-Review perch, expects `Code-Bouncer` `OnDone` `Drift-Verify` and `Drift-Verify` `OnDone` `Describe`.
`TestDriftRows_EntryGate` (tier0) builds the loom and darn sheds and asserts each Drift row's entry gate does not hold under the template setting and holds when the setting answers `drift.enabled` true; and, with the setting false and a status whose current row and `EnteredRow` are `Drift-Await` (a wait or halt that entered it), one step calls the row rather than recording it `Skipped`, as it does for `Drift-Verify`.
`TestDriftEntries` (tier0) is a table over the three entries: each builds with the kit's `Env`; an unknown key, a missing or unknown `subject`, a `capture` other than `detach` or `wait`, a `Drift-Verify` row with no `verify` gate or a second gate, and a nil `Drift` seam each refuse naming the way forward.

**ImpactSummary:** Two recipes gain three rows; the registry, `Env`, the loom wiring, the report's phase tables and the recipe and seam tests follow; batten builds unchanged.
