# Card 9 — fabric-exclude-list

**Edit:**
- `//internal/fabricengine/template.yaml`
- `internal/fabricengine#Config`
- `internal/fabricengine#ConfigSpec`
- `internal/fabricengine#wireJunctionSet`
- `internal/fabricengine#TestConfigTemplate`
- `internal/fabricengine_test#TestLoadConfig`

**Create:**
- `plan:internal/fabricengine#Config.WarpExcludePaths` -> `func (c Config) WarpExcludePaths() []string`
- `plan:internal/fabricengine#seedWarpExcludeLines` -> `func seedWarpExcludeLines(rec *Mutations, worktree string, paths []string) error`
- `plan:internal/fabricengine#SeedWarpExclude` -> `func SeedWarpExclude(l *lymcwd.Location) ([]string, error)`
- `plan:internal/fabricengine#ExcludeWayForward` -> `func ExcludeWayForward(path string) string`
- `plan:internal/fabricengine_test#TestSeedWarpExclude_SeedsWarpOnlyOnce` -> `func TestSeedWarpExclude_SeedsWarpOnlyOnce(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#mutateGitExclude`
- `internal/fabricengine#WiredNames`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine#Topology.AddLocal`
- `internal/fabricengine#LoadConfig`
- `internal/fabricengine#ConfigDirs`

**Intent:** Give fabric its list of warp paths to exclude and seed it (D5).

- `template.yaml` gains `warp_exclude: CLAUDE.local.md AGENTS.override.md .heddle`, with a comment saying it is a whitespace-separated list of paths relative to a warp worktree's root, excluded through the warp repository's `info/exclude`, separate from `pathspec` and never wired as a junction.
  The default reaches a hub through the template fill; no hub base file is written (PATTERN-sparse-hub-base).
- `Config` gains `WarpExclude string \`yaml:"warp_exclude"\``, and `WarpExcludePaths` splits it on whitespace.
  `ConfigSpec` classifies `warp_exclude` as `CategoryCrossPair`, since its line reaches every worktree of the repository, so the local layer refuses it; its comment says so for all three keys, and `config.go`'s file header, which names the branch prefix and the pathspec, names the exclude list beside them.
  `TestModules_EveryTemplateKeyClassified` already fails a template unit left unclassified, so it covers the new key unchanged.
- `seedWarpExcludeLines` appends an anchored `/<path>` line for each path to the `info/exclude` of the repository checked out at `worktree`, through `mutateGitExclude` (the exclude lock and atomic replace the junction seeding uses), skipping a line already present, and records `KindFileWritten` on the exclude file only when it changed.
  The lines are anchored at the repository root, not at the anchor subpath, because heddle writes at the warp root; no line is ever removed, since `info/exclude` is one file for every worktree of the repository.
- `wireJunctionSet` first loads the config through `LoadConfig(ConfigDirs(l))`, before `seedLymJunction`, so a load failure fails the wiring and writes nothing, as a junction-name load failure already does; it seeds `WarpExcludePaths()` into `WorktreePath(l, slug)` after the junction lines, so `Topology.Add`, `AddLocal`, the clone's wiring (`fabriccli.CloneAndWire` calls `WireJunctionsWith`) and `lym fabric reconcile`'s re-wire all seed it.
  Reconcile seeds the list only for a pair it re-wires, as D5's re-wires wording says: `repairPairWiring` reaches `wireJunctionSet` only when `checkJunctionHealth` reports the pair unhealthy, and a pair whose exclude lines alone were stripped gets them back at its next worktree event or `lym heddle render`, both of which seed through `SeedWarpExclude`.
  The list is never folded into the wired junction names, so health and drift checks never ask for a junction of it.
- `SeedWarpExclude(l)` loads the same config, seeds the list into `l`'s warp worktree and returns the paths; card 10's event builder calls it before every event, and through it `lym heddle render`.
  Weft is never seeded: nothing on the list is written in weft.
- `ExcludeWayForward(path)` returns the refusal text naming the path and its return to fabric's `warp_exclude` key through `lym config fabric`, so heddle's render (card 15) refuses a target off the list without spelling the key; `fabricengine/doc.go` notes it beside the exclude list.
- Tests:
  `TestConfigTemplate` asserts the template carries `warp_exclude` and resolves it to exactly `CLAUDE.local.md`, `AGENTS.override.md` and `.heddle`; its doc comment, its `//testtiming:keep` reason and `template_test.go`'s file header, which say "both keys", name the three keys.
  `TestLoadConfig` gains a row beside its local-pathspec refusal: a local `warp_exclude: x` is refused with `configengine.ErrInvalid` naming the local file and the key.
  `TestSeedWarpExclude_SeedsWarpOnlyOnce` (tier1, in a new file `internal/fabricengine/warpexclude_integration_test.go` with `//go:build tier1`) adds a pair to a `hubforge.CopyHub` hub and asserts the pair's warp `info/exclude` holds `/CLAUDE.local.md`, `/AGENTS.override.md` and `/.heddle` once each after `SeedWarpExclude` and `WireJunctions` run twice more; the weft repository's exclude holds none of them; `WiredNames` holds none of them; and after `Topology.Remove` of the pair the three lines are still there.
  Further rows: with the three lines stripped from `info/exclude` and one of the pair's junctions removed, `Topology.Reconcile` of the pair repairs the junction and seeds the lines again; a pair added with `Topology.AddLocal` has them; and with the hub's fabric config made unparseable, `WireJunctions` fails naming the config load and seeds nothing.

**ImpactSummary:** One new config key with a template default, one more exclude-seeding step inside `wireJunctionSet`, reached by every wiring caller, and the exported refusal text for a path off the list; no junction, health or drift behaviour changes.
