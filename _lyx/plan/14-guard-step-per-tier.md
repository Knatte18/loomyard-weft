# Card 14 — guard-step-per-tier

**Create:**
- `plan:internal/impactset#GuardTest` -> `type GuardTest struct`
- `plan:internal/impactset#FileTier` -> `func FileTier(file *ast.File) int`
- `plan:internal/impactset#GuardWrongTier` -> `const GuardWrongTier GuardScanKind = "wrong-tier"`

**Edit:**
- `internal/impactset#FindGuardTests` -> `func FindGuardTests(worktree string, dirs []string) (map[string][]GuardTest, error)`
- `internal/impactset#ModuleGuardTests` -> `func ModuleGuardTests(moduleDir string) (map[string][]GuardTest, error)`
- `internal/impactset#guardNamesInFile` -> `func guardNamesInFile(path, rel string) ([]GuardTest, error)`
- `internal/impactset#targetsFor` -> `func targetsFor(graph moduleGraph, changedPaths []string, guards map[string][]GuardTest) Targets`
- `internal/impactset#Targets`
- `internal/impactset#DeriveTargets`
- `internal/impactset#TestFindGuardTests`
- `internal/impactset#TestTargetsFor`
- `internal/impactset#TestModuleGuardTests_OverAModule`
- `internal/gatecli#runGuardStep` -> `func runGuardStep(ctx context.Context, out io.Writer, run tierRun, moduleDir string, guards map[string][]string, env []string, request testRequest) tierResult`
- `internal/gatecli#scope`
- `internal/gatecli#computeScope`
- `internal/gatecli#scanGuards`
- `internal/gatecli#planTierRuns`
- `internal/gatecli#tierRun`
- `internal/gatecli#guardRunOf`
- `internal/gatecli#runTest`
- `internal/gatecli#execution.runGuard`
- `internal/gatecli#guardScanRefusal`
- `internal/gatecli#planConfig`
- `internal/gatecli#scopesConfig`
- `internal/gatecli#scenarioConfig`
- `internal/gatecli#TestPlanTierRuns`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gateslot#Config`
- `internal/gateslot#configFile`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#decodeConfig`
- `internal/gateslot#MigrateConfig`
- `internal/gateslot/template.yaml`
- `internal/gateslot#TestMigrateConfig`
- `internal/gateslot#TestLoadConfig`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/impactset/doc.go`
- `internal/gatecli/doc.go`
- `internal/gateslot/doc.go`

**Delete:**
- `internal/impactset#compiledUnderIntegration`
- `internal/impactset#GuardNotIntegration`

**Intent:**
The guard step runs per tier (D6), so guard tests in `tier0` files run under `tier0`'s command once the tiers are disjoint, and `guard_tier` is retired.

`FileTier` reads a test file's `//go:build` constraint: the tier is the `N` of the `tierN` tag it names, and a file with none counts as `tier0`, so an untagged file and a transitional `tierN || integration` file (D13) both read right.
`GuardTest` holds a guard test's `Name` and its file's `Tier`; `guardNamesInFile`, `FindGuardTests`, `ModuleGuardTests` and `Targets.Guards` carry it.
The scan's `integration` rule is replaced: a `//lyx:guard` marker is allowed in a `tier0` or `tier1` file and refused in a `tier2` or `tier3` file as `GuardWrongTier`, naming the file and line; `guardScanRefusal`'s way forward for it is to move the marked test into a `tier0` or `tier1` file.
The input that reaches it is a `tier2` test file holding a marked `func Test…`; a `TestFindGuardTests` row reaches it, and another row asserts a marked test in a `tier1 || integration` file reads as tier 1.

In `gatecli`, the guard step groups the guard tests outside the scope by their file's tier and plans one guard run per group, each under that tier's command, slots, `timeout_min`, `prebuilt_lyx` and `dir`, after the listed tiers pass.
A group whose tier the call already ran whole, or carried whole (card 7), is skipped, and a group the green record holds under its own `-run` args carries.
A group whose tier is undeclared, or whose command lacks `{targets}` or `{args}`, is refused before any slot naming `lyx config gate`.
`runGuardStep` runs one group's `tierRun`; `guardRunOf` returns every guard run, and `runTest` runs them in tier order, stopping at the first failure.
With `guard_tier` gone, the guard scan runs in `tier0`'s `dir` (every hub declares `tier0`, which `planTierRuns` already requires), or the `-C` directory with explicit targets, as `computeScope` and `scanGuards` already set it for a narrowed computed scope; `runTest` runs the scan before planning the guard groups, since the groups come from the scan's tiers.

`guard_tier` is retired: `Config.GuardTier`, `configFile`'s field, the template line and its category go; `ConfigSpec` lists it among the retired keys, so the load drops it, and `MigrateConfig` prunes it, returning `guard_tier: retired, removed`.
`TestMigrateConfig` gains the acceptance row of a hub file holding `landing`, `final`, `round`, `card`, `batch` and `guard_tier`, reconciling to the D1 keys and scopes with no `guard_tier`, and a second pass changing nothing.
`TestPlanTierRuns` gains rows for guard tests in `tier0` and `tier1` files planning two groups under their own tiers, for a group skipped because its tier ran whole, for a group skipped because its tier was carried whole, and for a group refused because its tier is undeclared and because its tier's command lacks `{args}`.
`TestGateTest_Scenario` gains rows: a call running both groups; a repeat of the same call over the same tree carrying the guard groups under their `-run` args; and an `all` run after that guard pass on the same tree running the guard packages' tests, since a guard entry never covers an empty-args request.
The config seeds in the gatecli test helpers and `TestVerifyGate_Scenario` drop `guard_tier`.
The impactset and gatecli docs state the per-tier guard step and the new scan rule, the gatecli doc's Example block drops `guard_tier: 1` and writes its `gates` as keys, and the gateslot doc's Config section drops `guard_tier` and names it among the retired keys.

**ImpactSummary:** Re-signs the guard scan's results and the guard step to carry each test's tier and retires `guard_tier`; touches impactset, gatecli and gateslot together.
