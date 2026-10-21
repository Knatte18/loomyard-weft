# Card 2 — gate-tier-config

**Create:**
- `plan:internal/gateslot#Tier` -> `type Tier struct`
- `plan:internal/gateslot#Config.TierList` -> `func (c Config) TierList(gate string) ([]int, error)`
- `plan:internal/gateslot#Config.Tier` -> `func (c Config) Tier(number int) (Tier, error)`
- `plan:internal/gateslot#Config.GateCall` -> `func (c Config) GateCall(gate string, scope ...string) (string, error)`
- `plan:internal/gateslot#Config.TiersValue` -> `func (c Config) TiersValue() string`
- `plan:internal/gateslot#TestConfig_TierUse` -> `func TestConfig_TierUse(t *testing.T)`

**Edit:**
- `internal/gateslot#Config`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#decodeConfig` -> `func decodeConfig(data []byte, machineLayerApplies bool) (Config, error)`
- `internal/gateslot#TemplateConfig`
- `internal/gateslot#LoadConfig`
- `internal/gateslot#TestLoadConfig`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configcli#dirsOf`
- `internal/configcli#configDirs.selectedLayer`
- `internal/configcli#setModule`
- `internal/configcli#editOne`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#TestEditOne`
- `internal/configcli#TestDispatchSet`
- `internal/gateslot#TestTemplateConfigAndLimits`
- `internal/hubgeom#TestWebsterGeometry`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `internal/configcli/doc.go`

**Intent:**
Declare test tiers in the hub-wide `gate.yaml` (D1).

`Config` gains `Tiers map[string]Tier` (`tiers`, keyed `tier0`, `tier1`, ...), `Gates map[string][]int` (`gates`), `Scoper` (`scoper`, `go` or `none`), `GuardTier` (`guard_tier`), `RefuseRaw []string` (`refuse_raw`) and `TierParallel map[string]int` (`tier_parallel`, the machine override of a tier's `parallel`).
`Tier` holds `About`, `Command`, `Dir` (default `.`), `Env map[string]string`, `AllTargets`, `Parallel`, `TimeoutMin`, `Counter`, `Covers []int`, `Slots` (a count, or `all` read as 0 meaning every slot; default 1) and `PrebuiltLyx`, with the yaml keys D1 names.
`go_parallel` stays in this card; card 19 retires it.

`ConfigSpec` declares `tiers`, `gates` and `tier_parallel` as open maps, so `--set tiers.tier0={...}` writes one tier whole (card 1) and reconcile carries each whole.
`tier_parallel` is `CategoryMachine` beside `slots`, and every other new key is hub-file only: `tiers` is `CategoryExecutable`, since a tier command runs any shell command in a task worktree, and `gates`, `scoper`, `guard_tier` and `refuse_raw` are `CategoryGuard`.
The machine override is a sibling map rather than a path under `tiers`, because an open map is one unit in the layer merge, so a machine layer holding part of `tiers` would replace the hub's tiers whole;
`lyx config gate --layer machine --set tier_parallel.tier0=2` is the operator's form inside a hub, and the template comment says so, adding that a standalone worktree, which has no machine layer, sets the key in its own file with plain `lyx config gate`.

`decodeConfig` validates at load, each refusal naming the key and `lyx config gate`:
a declared tier lacking `command`, `timeout_min` or `counter`, or lacking `all_targets` when its command holds `{targets}`;
`parallel` and `timeout_min` below 1 (`parallel` defaults to 1 when absent), `slots` neither `all` nor at least 1;
a counter other than `go-json` or `vstest`, a scoper other than `go` or `none`;
a `covers` entry not lower than its tier's number, and a tier key not spelled `tier<N>`;
a `refuse_raw` entry that is not one or more words of letters, digits, `.`, `_` and `-` separated by single spaces.
A `tier_parallel` entry is read as an integer whether written plain or as the string scalar a scalar `--set` writes, and one that is no integer or is below 1 is refused naming the key and `lyx config gate`, leaving the layer to the clause `withConfigWayForward` appends from the files the load stacked, which names `--layer machine` only when the load stacked a machine file.
An entry for a tier the hub does not declare is dropped with a logged warning, so the load still succeeds; `LoadConfig` tells `decodeConfig` whether `configengine.LayerFile` reports the machine layer applies for the load's dirs, and when it does, the warning names both steps, plain `lyx config gate` for an entry in the hub file and the operator's `lyx config gate --layer machine` editor session for an entry in the machine file, since the stacked document cannot tell which layer holds it; when it does not, the warning names plain `lyx config gate` alone, where the key sits in the one worktree file.
`TemplateConfig` passes false, since the embedded template has no machine layer.
A declared tier's effective `Parallel` is its `tier_parallel` entry when one is set, from whichever layer sets it; the hub operator writes a tier's `parallel` inside the tier, and the template's comment on `tier_parallel` says it is the machine override.
`LoadConfig` outside every hub (no board told) with no worktree `gate.yaml` returns `ErrConfigAbsent` with a way forward naming `lyx config gate`, which writes the worktree's own file there, in place of `lyx fabric reconcile`, which only a hub has;
that way forward also says the worktree's `_lyx` directory must exist first and names creating it, since the write refuses `not initialized: _lyx/ directory not found` when it is absent.
So that step succeeds, configcli's `dirsOf` takes its board from `fabricengine.ConfigDirs(l).Board`, empty when the hub's board dir does not exist, in place of the unconditional `fabricengine.BoardDir`, and `configDirs.selectedLayer` maps an empty `--layer` on a hub-wide module to `LayerWorktree` when the dirs tell no board, the worktree-layer write path a per-worktree module already takes there, with no hub-file write, guard or commit, all of which run only for `LayerDefault`.
`setModule` and `editOne` then skip the fabric sync for a worktree-layer write when the dirs tell no board, since a worktree with no board has no weft to sync, and answer ok with the message `edited _lyx/config/<module>.yaml`; inside a hub the worktree-layer write of a per-worktree module keeps its sync.
The board dir is the one hub signal the dirs carry, so a hub whose board dir is absent or not statable also skips the sync, the same condition under which a hub-wide module's write already falls to the worktree file.
`internal/configcli/doc.go` says a worktree-layer write with no board runs no fabric sync.
So `lyx config gate` in a standalone worktree writes that worktree's own `gate.yaml`; inside a hub an empty `--layer` keeps selecting `LayerDefault`, and `--layer worktree` on a hub-wide module stays refused.
Cross-references are never checked at load, so the template and a hub whose gate lists name an undeclared tier both load, and every slot-only reader keeps loading.

`TierList(gate)` returns the gate's list, refusing, each naming `lyx config gate`: a gate name the config holds no list for, an empty list for any gate but `publish`, a listed tier the hub does not declare, and a hub with no `tier0`.
`Tier(n)` returns a declared tier or refuses naming `lyx config gate`.
Each of these refusals ends with the role clause `withConfigWayForward` carries, that a session that is not the prime takes none of these steps, reports status: FAILED and the orch runs them, since the sessions meeting them are mostly forks and writers.
`GateCall(gate, scope...)` renders the one call a site runs, `lyx gate test --wait --tier <n>...` in the list's order followed by the scope words, such as `--all`, `--base <rev>` or `--card <id>`; it returns `""` for an empty `publish` list.
`TiersValue()` renders the marker value prompts take in place of `verify_tiers` (D9): one line per declared tier in number order, `- tier<N>: <about>`, then one line per Go-read gate, `- gate <name>: [<n>, ...]`, or `none` when no tier is declared.

The template declares no tier, `gates` with `card`, `batch`, `round`, `final` and `landing` as `[0]` and `publish` as `[]`, `scoper: none`, `guard_tier: 0`, an empty `refuse_raw` and an empty `tier_parallel`, each with a comment stating the key's rule.
`doc.go`'s Config section documents the keys, the load-time and use-time refusals and the machine override.

Tests: `TestLoadConfig` gains one row per load refusal above, a row loading a hub file whose gate list names an undeclared tier, a row loading a tier written in flow form with `parallel` and `slots` absent and read as 1, a row for a machine `tier_parallel` entry naming an undeclared tier, which loads and drops the entry, beside one whose entry overrides a declared tier's `parallel` and one whose entry is the string scalar a scalar `--set` writes, and a standalone row with no worktree `gate.yaml` refused naming `lyx config gate` and the `_lyx` clause.
The drop's warning is asserted in a subtest that captures `logger` output, not parallel, naming that process-global state, for the machine entry, whose warning names both plain `lyx config gate` and `lyx config gate --layer machine`, and for a standalone row whose worktree-file `tier_parallel` entry names an undeclared tier, which drops with a warning that names only plain `lyx config gate`, with no `--layer machine`.
`TestTemplateConfigAndLimits` asserts the template's new defaults.
hubgeom's `TestWebsterGeometry` makes its first `Limits()` assertion before it creates the board directory, so `fabricengine.ConfigDirs` tells no board and the no-board way forward applies: it expects `lyx config gate` and the `_lyx` clause in place of `lyx fabric reconcile`.
`TestDispatchLayers` gains the row where `lyx config gate --set` in a standalone worktree with no board writes that worktree's own `gate.yaml`, which a following `gateslot.LoadConfig` over the same dirs reads, beside the same write through the editor; both rows assert an ok envelope and that the sync is never called.
`TestEditOne` and `TestDispatchSet` build their dirs with no board, so their sync rows move to the hub case that keeps its fabric sync: `TestEditOne`'s `valid edit is synced and reported` and `failing sync is reported with its output` rows pass `editOne` `configDirs` whose `board` is an existing board directory, as `newHubFixture` builds, since `worktreeDirs` leaves `board` empty; `TestDispatchSet`'s two `wantSyncs: 1` rows first create the board directory of `makeLayoutAt`'s hub, which `fabricengine.ConfigDirs` then tells; the no-board case is `TestDispatchLayers`' new rows.
`TestModules_EveryTemplateKeyClassified` pins `tier_parallel` beside `go_parallel` and `slots` as the gate module's machine keys.
The new `TestConfig_TierUse` covers the use-time surface no existing test reaches: `TierList` for a declared list, a gate name the config holds no list for, a dangling entry, an empty non-`publish` list, an empty `publish` list and a missing `tier0`, each refusal ending with the role clause; `Tier` for an undeclared number; `GateCall` for a two-tier hub, for an empty `publish` list returning `""`, and for `final` and `landing` holding the same list rendering the same call, the key Merge-In's skip on a tree Webster's gate passed rests on; `TiersValue` for a two-tier hub and for the template.

**ImpactSummary:** additive fields and methods on the gate config; every existing reader of `slots`, `go_parallel` and `busy_quit_sec` loads unchanged, and the template gains keys reconcile seeds into each hub file; `lyx config` with no `--layer` on a hub-wide module in a standalone worktree now writes the worktree's file instead of answering that the hub layer does not apply, and every worktree-layer write with no board, a per-worktree module's included, now answers ok with no fabric sync instead of an error envelope from the failed sync.
