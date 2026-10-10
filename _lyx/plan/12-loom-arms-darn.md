# Card 12 — loom-arms-darn

**Create:**
- `plan:internal/loomcli#resolveSeedRecipe` -> `func resolveSeedRecipe(existing shedrun.Seed, found bool) string`
- `plan:internal/loomcli#loomCLI.resolveRecipe` -> `func (c *loomCLI) resolveRecipe(location *lyxcwd.Location) error`
- `plan:internal/loomcli#loomCLI.newShed` -> `func (c *loomCLI) newShed() (*shedengine.Shed, error)`
- `plan:internal/loomcli#loomCLI.recipeRouting` -> `func (c *loomCLI) recipeRouting() (shedengine.Routing, error)`
- `plan:internal/loomcli#reworkRowFor` -> `func reworkRowFor(recipe string) string`
- `plan:internal/loomcli#darnVerifySource` -> `func darnVerifySource(l *lyxcwd.Location) verifySource`
- `plan:internal/loomcli#darnMergeBase` -> `func darnMergeBase(l *lyxcwd.Location) func() (string, error)`
- `plan:internal/loomcli#latestDarnOutcome` -> `func latestDarnOutcome(statusPath, statusLockPath string) (loomshed.DarnOutcome, bool, error)`
- `plan:internal/loomcli#darnDeps` -> `func darnDeps(l *lyxcwd.Location, verifyDir string) loomshed.DarnDeps`
- `internal/loomcli/wiring_darn_test.go`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.armAt`
- `internal/loomcli#loomCLI.loadRouting`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/loomcli#loomSeedFor` -> `func loomSeedFor(recipe, parent, driver string) shedrun.Seed`
- `internal/loomcli#loomCLI.seedAndCommitBootstrap`
- `internal/loomcli#rejectDeps`
- `internal/loomcli#rejectVerb`
- `internal/loomcli#loomCLI.rejectCmd`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli/arm_seed_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/sharedbootstrap_test.go`
- `internal/loomcli/reject_test.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D1, D2's loom side, D6, D7 and D12's reject help: loom's arm drives a darn run from the seed's recipe.
`loomCLI` gains `recipe string`; `armAt` fills it through `resolveRecipe` right after `resolveRunID`, reading the seed at the run-id and taking `shedrun.RecipeLoom` when none exists, so every verb, the lightweight ones included, knows the recipe (`PATTERN-driver-choice-single-site` is untouched: only the recipe field is read).
`recipeRouting` returns `loomrecipe.Routing(budget)` for loom and `loomrecipe.DarnRouting()` for darn, and `loadRouting` uses it;
`newShed` returns `loomrecipe.New` or `loomrecipe.NewDarn` over `c.env` and `c.shedPaths`, and `specFor`'s `run` and `buildLoomShed`'s tail call it, so `lyx shed step`, `lyx loom resume` and `lyx loom goto Darn` over a darn seed build the darn graph.
`wire` branches on the recipe after the loom config loads: for `darn` it loads `darnengine.LoadConfig` at the board dir strictly (a failure carries `hubConfigWayForward`), calls `darnengine.VerifyCommand` once so `start`, `run`, `step` and `resume` refuse an empty `verify` early with its message, and fills `Env.VerifyCommand` from `darnVerifySource` (the reader closure and the clause `run "lyx loom goto --to Darn", then "lyx loom resume", in the task worktree; the Darn spawn renders the Publish failure record and its gate runs the failing tests`), `Env.VerifyMergeBase` from `darnMergeBase` (`fabricengine.Open` then `Fabric.MergeBase` of the recorded parent branch, at call time), `Env.DarnVerifyAttempts` from the config, `Env.Darn` from `darnDeps` (rejection read and clear over `loomengine.LoomRejectionPath`, a landing-directory commit like `CommitDescription`, `latestDarnOutcome` over the status file, and the Publish failure record's presence through `verifytree.ReadPublishFailure`), and `Env.DarnSpec` as a closure building `darnengine.DarnInputs` from the location, the stencils and specs dirs, the friction dir, the parent name, the slug, the description path and the told tokens;
for `loom` the verify source stays `planVerifySource` and the darn seams stay nil.
`loomPreRun` and `buildLoomShed` pass the recipe's verify source to `landingDeps`, so Publish's and Finalize's verify and failed-verify clause follow the recipe.
`loomSeedFor` takes the recipe, and `seedAndCommitBootstrap` passes `resolveSeedRecipe`'s answer, the recorded seed's recipe or `loom` when none exists, so `lyx loom start` never writes a recipe over an existing seed.
`rejectDeps` gains `reworkRow string` and `rejectionPending func() (bool, error)`; `rejectVerb` accepts a run awaiting or blocked at `PR-Gate`, or blocked at the rework row, where a run whose rework row is `Darn` also needs a pending rejection record, since it has no pull request to reject otherwise, and the refusal names the recipe's two rows;
`rejectCmd` fills them through `reworkRowFor` and the recipe's routing, and its help names both recipes' rework rows: in loom the next start sends the findings to `PR-Rework`, which archives the plan generation, and in darn to a fresh `Darn` spawn.
Tests: `wiring_darn_test.go` wires a darn seed over the hand-built hub fixture and asserts the darn seams, the verify source, the attempts and the empty-verify refusal with its way forward, and that a loom seed leaves the darn seams nil;
`arm_seed_test.go` gains rows for the recipe resolution, `sharedbootstrap_test.go` for the preserved recipe, `reject_test.go` for the darn rows, and `wiring_test.go` for the routing per recipe.
The refusal spec gains the row `darn.yaml: verify missing or empty` and the `Publish: verify failed` row's way forward gains darn's clause;
the overview's loom bullet names the darn recipe, its selection from the seed and the verbs that serve it.

**ImpactSummary:** Re-signs `loomSeedFor` with its two callers here; every loom verb reads the seed's recipe once at arming, and loom seeds behave as before.
