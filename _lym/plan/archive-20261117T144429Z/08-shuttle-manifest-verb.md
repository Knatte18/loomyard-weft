# Card 8 — `lym shuttle manifest` prints each engine's manifest for a seat

**Create:**
- `plan:internal/shuttleparity#ManifestReport` -> `type ManifestReport struct`
- `plan:internal/shuttleparity#Report` -> `func Report(env SeatEnv, name string, engines shuttleengine.EngineSet, cfg shuttleengine.Config) (ManifestReport, error)`
- `plan:internal/shuttleparity#LookupSeat` -> `func LookupSeat(seats []Seat, errs []error, name string) (Seat, error)`
- `plan:internal/shuttlecli#shuttleCLI.manifestCmd` -> `func (c *shuttleCLI) manifestCmd() *cobra.Command`
- `plan:internal/shuttleparity#TestReport` -> `func TestReport(t *testing.T)`
- `plan:internal/shuttleparity#manifestEngine` -> `type manifestEngine struct`
- `plan:internal/shuttlecli#TestManifestCmd` -> `func TestManifestCmd(t *testing.T)`

**Edit:**
- `internal/shuttlecli#shuttleCLI`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#TestCommand_PositionalArgValidation`

**Extends:**
- `internal/shuttleparity/doc.go`
- `//docs/overview.md`

**Uses:**
- `plan:internal/shuttleparity#PrepareEach`
- `plan:internal/shuttleparity#Compare`
- `plan:internal/shuttleparity#Seats`
- `plan:internal/shuttleparity#Unmet`
- `plan:internal/shuttleparity#UnrunHooks`
- `plan:internal/shuttleparity#ConfiguredSandbox`
- `plan:internal/shuttleengines#NewWithSeams`
- `internal/shuttlecli#TestOpenCmd_ListModels`

**Intent:**
D10's verb.

`Report(env, name, engines, cfg)` makes a temporary directory with `lymtemp.MkdirTemp` and sets it as `env.Dir`, so the stand-in inputs `Seats` writes and every `Prepare` run land inside it; it removes the directory before it returns, on every path.
It builds the catalog with `Seats`, picks the seat with `LookupSeat`, renders the seat's spec for each engine of the set, and runs `PrepareEach` into the directory.
It returns `ManifestReport{Seat, Manifests map[string]shuttleengine.Manifest, Missing []shuttleengine.ManifestItem, Unmet map[string][]shuttleengine.Requirement}`, `Missing` from `Compare` and `Unmet` naming each engine that cannot start the seat.
`LookupSeat` returns the seat whose `Name` is name.
An unknown name is refused with the way forward naming the catalog's seat names.
A name whose builder failed is refused naming that builder's error, so a failing builder refuses only its own seats.

`shuttleCLI` gains a field `manifestEnv func() (shuttleparity.SeatEnv, shuttleengine.EngineSet, error)`, set by `PersistentPreRunE`.
It returns the seat environment of the current worktree: the hub's stencils and specs directories, the worktree's layout, config dirs and model registry, with the engine set from `shuttleengines.NewWithSeams(c.cfg, c.configDirs, LaunchSeams{RunHook: shuttleparity.UnrunHooks, SandboxProbe: shuttleparity.ConfiguredSandbox})`.
`lym shuttle manifest <seat>` (audience operator, one required positional argument) calls it and then `Report`.
- With nothing missing and no unmet requirement it prints `output.Ok` with both manifests and the verdict, plugin contexts listed unrun.
- With an item missing it prints `output.Err` naming each missing item as `<channel> <key>`, with the report, and exits non-zero, with the way forward: give the Codex engine a delivery for each missing item in `internal/shuttleengine/codexengine`, and run no Codex bench of this seat until the verb passes.
- With an engine that cannot start the seat it prints `output.Err` naming the engine and its unmet requirements, with the report, and exits non-zero, since that engine's `Start` refuses the seat.
`Long` states the bounds: it starts no session and runs no plugin hook or sandbox probe;
it writes nothing outside its temporary directory but the Codex home a Codex `Prepare` readies (its links, trust record and `config.toml`), and keeps no manifest copy, since it never calls `Runner.launch`;
plugin contexts are listed by plugin and hook unrun, and only a run's record says whether one was delivered;
the developer-instructions size check leaves out plugin contexts, so a seat it passes can still be refused at a real launch.
`Command` adds `manifestCmd`, and its doc comment names the verb; the verb takes the strict cwd gate of every other verb.
`shuttleparity`'s `doc.go` names `Report`, and `docs/overview.md` adds the verb to its shuttle verb list and `internal/shuttleparity` to its package tree.

Tests, tier0:
- `TestReport` is a table over a fake engine set of two `manifestEngine` fakes, whose `Prepare` writes a given manifest, on a catalog of one fixture table: matching manifests give no `Missing`; a Codex manifest lacking a compared item gives exactly that item; a Codex plugin context omitted as unrun gives none;
  a Codex fake meeting no requirement gives `Unmet` for a forks seat; an unknown seat name is refused naming the seats;
  a fake whose `Prepare` fails returns an error naming the engine, and one that writes no record returns the error naming the engine and `shuttleengine.WriteManifest`;
  after each row the temporary directory is gone, so no `manifests` copy survives anywhere.
- `TestManifestCmd` builds `shuttleCLI` by hand with a `manifestEnv` over the fake set, as `TestOpenCmd_ListModels` builds its receiver, and runs `manifestCmd`: a matching seat prints `ok` with both manifests and exits zero; a seat with a missing Codex item prints the error naming `<channel> <key>` and exits non-zero; an unknown seat prints the error naming the catalog's seats.
- `TestCommand_PositionalArgValidation` gains a row: `lym shuttle manifest` with no argument is refused by the positional check.

**ImpactSummary:** One new verb in shuttle's subtree; `shuttleCLI` gains one field and `Command` one `AddCommand` entry; no existing verb changes.

**Commit:** 8: Add lym shuttle manifest
