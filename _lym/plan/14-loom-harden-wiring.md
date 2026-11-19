# Card 14 — loom-harden-wiring

**Create:**
- `plan:internal/loomcli#hardenReviewSegment` -> `const hardenReviewSegment = "Harden-Review"`
- `plan:internal/loomcli#hardenSegmentSettings` -> `func hardenSegmentSettings(cfg hardenshed.Config, loomCfg loomengine.Config, registry modelspec.Registry) (shedrecipe.SegmentSettings, error)`
- `plan:internal/loomcli#hardenCampaign` -> `func hardenCampaign(location *lymcwd.Location, cfg hardenshed.Config, settings shedrecipe.SegmentSettings) *hardenshed.Campaign`
- `plan:internal/loomcli#HardenReapDeps` -> `func HardenReapDeps(location *lymcwd.Location, row string, point hardenshed.ReapPoint) (hardenshed.ReapDeps, error)`
- `plan:internal/loomcli#hardenLiveArgs` -> `func hardenLiveArgs(key string, copies int, packages []string) []string`
- `plan:internal/loomcli#hardenLiveEnv` -> `func hardenLiveEnv(env []string) []string`
- `plan:internal/loomcli#withHardenBudget` -> `func withHardenBudget(budgets map[string]int, recipe string, dirs configengine.Dirs) (map[string]int, error)`
- `plan:internal/loomcli#recipeTrees` -> `func recipeTrees(l *lymcwd.Location, recipe string) func() (map[string]string, error)`
- `plan:internal/loomcli#TestWire_HardenerRecipe` -> `func TestWire_HardenerRecipe(t *testing.T)`
- `plan:internal/loomcli#TestReviewBudget_HardenReview` -> `func TestReviewBudget_HardenReview(t *testing.T)`
- `plan:internal/loomcli#TestHardenLiveArgs` -> `func TestHardenLiveArgs(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireTreeScope`
- `internal/loomcli#loomCLI.reviewBudget`
- `internal/loomcli#loomCLI.verifySource`
- `internal/loomcli#darnTreeScope` -> `func darnTreeScope(l *lymcwd.Location, trees func() (map[string]string, error)) func(gateKey string) (string, bool, error)`
- `internal/loomcli#darnVerifySource` -> `func darnVerifySource(l *lymcwd.Location, trees func() (map[string]string, error)) verifySource`
- `internal/loomcli#perchHardCap`
- `internal/loomcli#TestPerchHardCap`
- `internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomcli#loomSegmentSettings`
- `internal/loomcli#loomSegmentBudgets`
- `internal/darnengine#TreeScopedTargets`
- `internal/reviewbudget#Route`

**Intent:**
Wire the Harden perch and a `hardener` run's seams in loom (D1, D3, D4, D9).

For a run whose recipe holds a `HardenFix` row, `loomCLI.wire` loads the hub's `hardener.yaml` strictly through `hardenshed.LoadConfig`, its error naming its own way forward (card 3), and:
- Adds `Env.Segments[hardenReviewSegment]` from `hardenSegmentSettings`: the reviewer and fixer lists expanded by `hardenshed.Rotate` to `hard_max_bounces` and resolved through the registry, keys naming `hardener.yaml`; the Bouncer judge and seed from `hardener.yaml`'s `judge` and `seed`, falling back to loom.yaml's resolution when empty; `MaxBounces` and `HardMaxBounces` from `max_bounces` and `hard_max_bounces`, `ConfigFile` `hardener.yaml`. So round n takes entry n modulo the list's length up to the hard cap, a round a parent's continue adds included.
- Fills `Env.Harden` with `hardenCampaign`, whose seams are:
  - the live `Run`: spawns `lym` with `hardenLiveArgs` in the worktree through `selfexec.LymBinary`, its environment `hardenLiveEnv`, logged at its spawn and waited on. `hardenLiveArgs` is `gate test --wait --gate <key>`, `--copies <n>` when n is above 0, then the packages, and names no tier. `hardenLiveEnv` drops `gateslot.InheritEnv`, so the call is top-level and acquires its own slots, never running inside a slot the row holds.
  - the reap: `HardenReapDeps`, the one builder of the reap's production seams: the sandbox root of the location's worktree, `proc.ListProcesses` (nil off Linux), `runproc.Owners` and `runproc.End` over each sandbox hub's `fabricengine.RunProcDir`, `runproc.EndProcess`, `reedengine.StopServer` with reed's tmux path, and this process's pid. Cards 15 and 16 call it too.
  - `Head` and `Commits`: the fabric handle's `HeadSHA` and `CommitLog` from the row's start head to HEAD.
  - the archive `Commit`: `fabricengine.CommitAnchoredPaths`.
  - the models: the segment's resolved lists, labelled per round.
- Fills `Env.PreflightHarden` with `hardenshed.CheckCampaign` over the loaded config, the tiers each live key's run tests from the gate config (the key's tiers with each tier's `covers`), `gateslot.HubSetsGate` for each key, `impactset.HoldsTierTest`, and `Route` over `reviewbudget.Route` with `hardenshed.ConfigSpec`.
Every other recipe leaves `Env.Harden` and `Env.PreflightHarden` nil and adds no Harden segment.

`withHardenBudget` adds the Harden-Review budget, `hardener.yaml`'s `max_bounces`, to a budget map when the recipe holds the segment, falling back to the template's value with a Warn when the file does not load, as `reviewBudget` falls back for loom.yaml.
`loomCLI.reviewBudget` returns its map through it, and so does the `SegmentBounces` routing in `wire`, so `lym loom status` on a hardener run reports `max_bounces` as the Harden-Review budget.
`perchHardCap` answers the Harden-Bouncer row from `hardener.yaml`'s `hard_max_bounces`, its raise routed through `hardenshed.ConfigSpec`, before the `loomPerches` lookup.

`darnTreeScope` and `darnVerifySource` take the tree map's reader in place of reading `darn.yaml` themselves.
`recipeTrees` reads `hardener.yaml`'s `tree_scope` for a recipe holding a `HardenFix` row and `darn.yaml`'s otherwise, at each call.
`loomCLI.verifySource` and `wireTreeScope` pass it.
For a hardener run, `wireTreeScope` loads `hardener.yaml` in place of `darn.yaml`, sets `Env.DarnVerifyAttempts` from its `verify_attempts` and sets no verify budget, so changing `darn.yaml` leaves a hardener run's verify scope and attempts unchanged.
The `verify` gate of Harden-Burler-Fix runs the `harden-burler-fix` key over the touched trees' targets for a round diff wholly under those trees.

Tests:
- `TestWire_HardenerRecipe` is tier1 over a hub fixture with a hardener seed:
  - the Harden segment's settings come from `hardener.yaml`; with a three-entry list and a hard cap of 10, `Models.Pick` gives rounds 1 to 9 entries 1, 2, 3, 1, 2, 3, 1, 2, 3;
  - `Env.Harden` and `Env.PreflightHarden` are set;
  - with `hardener.yaml`'s `tree_scope` naming a tree and a committed diff wholly under it, `Env.VerifyTreeScope("harden-burler-fix")` answers the key's call over that tree's target;
  - editing `darn.yaml`'s `tree_scope` and `verify_attempts` leaves the hardener verify tree scope and the Harden verify attempts unchanged.
- `TestHardenLiveArgs` is tier0: the harden-live call carries `--wait --gate harden-live` and the packages with no `--copies` and no `--tier`; the copies call carries `--copies <n>`; `hardenLiveEnv` drops `LYM_GATE_SLOT` and keeps every other variable.
- `TestReviewBudget_HardenReview` is tier0: the projected Harden-Review budget is `max_bounces`, and the template's when the file does not load.
- `TestPerchHardCap` gains the Harden-Bouncer row.
- `TestDarnTreeScope_ClassifiesAtEachCall` passes the darn reader.

**ImpactSummary:** loom's wiring gains a hardener branch keyed on the HardenFix engine and one exported reap-seam builder, and the two darn tree-scope readers take their tree source as an argument; darn and tapestry runs read the same files as before.
