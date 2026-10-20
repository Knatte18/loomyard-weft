# Card 3 — lenses module

**Create:**
- `internal/lenses/template.yaml`
- `internal/lenses/doc.go`
- `plan:internal/lenses#configTemplate` -> `var configTemplate string`
- `plan:internal/lenses#MaxFanLenses` -> `const MaxFanLenses = 16`
- `plan:internal/lenses#Config` -> `type Config struct`
- `plan:internal/lenses#Lens` -> `type Lens struct`
- `plan:internal/lenses#ConfigTemplate` -> `func ConfigTemplate() string`
- `plan:internal/lenses#ConfigPerNameMaps` -> `func ConfigPerNameMaps() []string`
- `plan:internal/lenses#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/lenses#Load` -> `func Load(dirs configengine.Dirs) (Config, error)`
- `plan:internal/lenses#ResolveFan` -> `func ResolveFan(cfg Config, name string) ([]Lens, error)`
- `plan:internal/lenses#CheckLoad` -> `func CheckLoad(dirs configengine.Dirs) error`
- `plan:internal/lenses#FanList` -> `type FanList []string`
- `plan:internal/lenses#FanList.UnmarshalYAML` -> `func (l *FanList) UnmarshalYAML(node *yaml.Node) error`
- `plan:internal/lenses#FanShapeError` -> `type FanShapeError struct`
- `plan:internal/lenses#FanShapeError.Error` -> `func (e *FanShapeError) Error() string`
- `plan:internal/lenses#FanKey` -> `type FanKey struct`
- `plan:internal/lenses#ValidateFanKeys` -> `func ValidateFanKeys(dirs configengine.Dirs, module string, keys []FanKey) error`
- `plan:internal/lenses#writeLensesYAML` -> `func writeLensesYAML(t *testing.T, baseDir, contents string)`
- `plan:internal/lenses#TestLoad` -> `func TestLoad(t *testing.T)`
- `plan:internal/lenses#TestResolveFan` -> `func TestResolveFan(t *testing.T)`
- `plan:internal/lenses#TestConfigTemplate_DecodesAndIsSelfConsistent` -> `func TestConfigTemplate_DecodesAndIsSelfConsistent(t *testing.T)`
- `plan:internal/lenses#TestCheckLoad` -> `func TestCheckLoad(t *testing.T)`
- `plan:internal/lenses#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`
- `plan:internal/lenses#TestMain` -> `func TestMain(m *testing.M)`

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configreg#fingerprintOf`
- `internal/configreg#TestModules_SeedOnlyAndHubWideFlags`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestFingerprint`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#TestPrintResolved`
- `cmd/lyx#configStrictnessDegradingSet`
- `pattern/PATTERN-leaf-packages.md`
- `pattern/PATTERN-config-strictness.md`
- `docs/overview.md`

**Uses:**
- `internal/configengine#LoadOrTemplate`
- `internal/testkit/scankit#AssertImportAllowlistNoStale`
- `internal/testkit/logcapture#CaptureVerbose`
- `internal/configcli#newHubFixture`
- `internal/configcli#seedModuleConfig`

**Intent:** Create the `lenses` config module in a new leaf package `internal/lenses` (D1) and register it, beside `burler`, which card 8 retires.

- `template.yaml` is today's `internal/burlerengine/template.yaml` with two changes.
  Its `twill-discussion` fan becomes `interactions`, `existing-paths`, `bound-breaker`, `walkthrough`, `goal-scope`, `board-fit`, `leanness`, in that order (D5); `premises` stays defined, and no lens is added or removed.
  Its header comment names the file `lenses.yaml` and is rewritten to D2's reconcile and D3's load: reconcile adds each template lens or fan the file lacks and keeps every lens and fan the file holds, prose and fan lists unchanged, so a later template edit reaches the file only once its entry is deleted from the file or edited in;
  a lens or fan the template lacks is kept and reported, never pruned;
  a template lens or fan cannot be removed, since a deleted one resolves to the template's value at the next load and is written back at the next reconcile, and a fan is turned off at the loom or darn fan key entry that names it, by pointing it at another fan or setting it to `""`.
  The lens and fan authoring guidance below the header stays.
- `ConfigSpec` names the module `lenses`, carries the template, `PerNameMaps: ConfigPerNameMaps()` (`lenses` and `fans`), and classifies both keys `CategoryGuard`, so no local layer and no run setting holds them; it is not `HubWide`.
- `Load` reads through `configengine.LoadOrTemplate` with `ConfigSpec()` and decodes the resolved bytes strictly into `Config` (`KnownFields`), each error prefixed `lenses: ` (D3).
  An absent base file resolves the template, and a lens or fan the stack lacks resolves to its template value through the load's fill, so `ResolveFan` reads the loaded `Config` alone with no per-name template fallback.
- `ResolveFan` resolves a fan name into its ordered `[]Lens`, preserving order and repeats, and refuses an unknown fan naming the known fans, an empty fan, a fan longer than `MaxFanLenses` and a fan naming an undefined lens, each naming the fan.
- `CheckLoad` runs `Load`, then `ResolveFan` over every fan of the loaded config in name order, and returns every failure joined in that order, so its text is deterministic for `lyx config lenses --set`'s before-and-after comparison.
- `FanList`, `FanShapeError` and `FanKey` are burlerengine's fan key grammar, moved as they stand; `ValidateFanKeys` keeps its behaviour, reads the config through `Load` and resolves through `ResolveFan`, and its refusals name `lenses.yaml` (`fix lenses.yaml, or set the key to empty to run the segment solo`; `define the fan in lenses.yaml, or set the entry to ""`).
- `doc.go` documents the module: the file, the per-name reconcile and its bound, the guard keys, the load stance, fan resolution and the fan key grammar loom and darn share.
- `internal/lenses` imports only the standard library, `internal/configengine` and `gopkg.in/yaml.v3`; `TestLeafInvariant_AllowlistOnly` pins that with `scankit.AssertImportAllowlistNoStale`, and `TestMain` runs the package under `tmuxkit.Main` as its siblings do.
- `configreg.Module` gains `PerNameMaps []string`, documented beside `OpenMaps`; `fingerprintOf` hashes it after the open maps; `Modules` registers `{Name: "lenses", Template: lenses.ConfigTemplate, PerNameMaps: lenses.ConfigPerNameMaps(), CheckLoad: lenses.CheckLoad, Spec: lenses.ConfigSpec}` in sort order.
- `PATTERN-leaf-packages`'s background gains an `internal/lenses` section with its import set; `PATTERN-config-strictness`'s background lists `lenses` among the degrading callers, and `configStrictnessDegradingSet` gains `internal/lenses`.
- `docs/overview.md`'s package tree gains the `internal/lenses/` line.

Tests:
`TestConfigTemplate_DecodesAndIsSelfConsistent` and `TestResolveFan` are burlerengine's, moved.
`TestConfigTemplate_DecodesAndIsSelfConsistent` loads through `Load` over dirs holding no `lenses.yaml`, so it owns the criterion that a load with no file resolves every template fan; it pins `twill-discussion` to D5's seven lenses in order and its entry count from 6 to 7, and keeps its `//testtiming:keep` marker and reason.
`TestResolveFan` keeps the unknown-fan, empty-fan, over-`MaxFanLenses` and undefined-lens refusal rows.
`TestLoad` rows: a board-default `lenses.yaml` lacking a template lens resolves a fan naming that lens with the template's prose; a hub-only lens loads and logs no unknown-keys warning (captured with `logcapture.CaptureVerbose`); a worktree-layer lens replaces the default's per name; a local-layer `lenses` entry fails naming the local file; an unknown top-level key is refused with the `lenses: ` prefix; a lens prose holding a bare `$` loads verbatim; a lens prose holding `${env:NAME}` with `NAME` unset and no default fails the load (D3's bound).
`TestLoad`'s rows run serially, since `logcapture.CaptureVerbose` swaps the process-global logger, as `TestLoad_LayerStack`'s do.
`TestCheckLoad` rows: the template passes; a file with two broken fans returns both failures in name order; a malformed `lenses.yaml` returns the load's error alone and resolves no fan.
The fan key grammar and `ValidateFanKeys` move with no test of their own: loom's `TestLoadConfig_Loads` and `TestLoadConfig_Refuses` and darn's `TestLoadConfig` reach their shape refusals, empty lists, never-read file, malformed file and unknown fan through the loaders, and cards 6 and 7 retarget those rows to `lenses.yaml`.
`TestModules_SeedOnlyAndHubWideFlags` also checks `spec.PerNameMaps` equals the entry's `PerNameMaps`;
`TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` gains `"lenses": {"fans", "lenses"}`;
`TestFingerprint` gains an "added PerNameMaps entry" row that changes the fingerprint.
`TestDispatchLayers` gains `lenses` rows on the default layer, the hub's lens file, with the board seeded with the template:
a `--set` of `lenses.mine=<prose>` and `fans.mine=[mine]` succeeds, commits once, and the board's stack resolves `mine` through `lenses.Load` and `lenses.ResolveFan` to that prose;
a `--set` leaving a fan empty (`fans.standard=[]`) is refused through the load check `CheckLoad` gives the hub-wide save and leaves the board's `lenses.yaml` byte-identical.
The other two refusals the criterion names, a fan naming an undefined lens and one over the cap, reach the same rollback through `CheckLoad`'s fan resolution, whose kinds `TestResolveFan` and `TestCheckLoad` pin.
The guard classification that closes the local layer is pinned by `TestModules_EveryTemplateKeyClassified`'s `lenses` entry.
`TestPrintResolved` gains a subtest printing `lenses` over a board default seeded with the template, asserting a template lens and the `twill-discussion` fan are annotated `# default`.

**ImpactSummary:** Adds one registered module and one Module field; configreg's fingerprint and module list change, burler and every consumer stay untouched, and configsync still reconciles lenses without per-name handling until card 4.
