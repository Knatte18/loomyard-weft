# Card 8 — `lym shuttle manifest` prints each engine's manifest for a seat

**Create:**
- `internal/shuttleparity#ManifestReport`
- `internal/shuttleparity#Report`
- `internal/shuttleparity#report`
- `internal/shuttleparity#LookupSeat`
- `internal/shuttlecli#shuttleCLI.manifestCmd`
- `internal/shuttleparity#TestReport`
- `internal/shuttlecli#TestManifestCmd`

**Edit:**
- `internal/shuttlecli#shuttleCLI`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#TestCommand_PositionalArgValidation`
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/shuttleparity/doc.go`
- `//docs/overview.md`

**Uses:**
- `internal/shuttleparity#PrepareEach`
- `internal/shuttleparity#Compare`
- `internal/shuttleparity#seatsOf`
- `internal/shuttleparity#seatTables`
- `internal/shuttleparity#seatTable`
- `internal/shuttleparity#manifestEngine`
- `internal/shuttleparity#Unmet`
- `internal/shuttleparity#UnrunHooks`
- `internal/shuttleparity#ConfiguredSandbox`
- `internal/shuttleengines#NewWithSeams`
- `internal/shuttlecli#TestOpenCmd_ListModels`

**Intent:**
D10's verb.

`Report(env, name, engines, cfg)` is `report(env, seatTables, name, engines, cfg)`; `report` takes the catalog as an argument so `TestReport` runs on fixture tables.
`report` makes a temporary directory with `lymtemp.MkdirTemp` and sets it as `env.Dir`, so the stand-in inputs `seatsOf` writes and every `Prepare` run land inside it; it removes the directory before it returns, on every path.
It builds the catalog with `seatsOf`, picks the seat with `LookupSeat`, renders the seat's spec for each engine of the set, and runs `PrepareEach` into the directory.
It returns `ManifestReport{Seat, Manifests map[string]shuttleengine.Manifest, Missing []shuttleengine.ManifestItem, Unmet map[string][]shuttleengine.Requirement}`, `Missing` from `Compare` and `Unmet` naming each engine that cannot start the seat.
`LookupSeat` returns the seat whose `Name` is name.
A name no built seat holds is refused naming the catalog's seat names and every builder error `seatsOf` returned, each with card 6's way forward, so a seat missing because its builder failed points at that builder's stand-ins.

`shuttleCLI` gains a field `manifestReport func(name string) (shuttleparity.ManifestReport, error)`, set by `PersistentPreRunE`.
Its production closure builds the seat environment of the current worktree (the hub's stencils and specs directories, the worktree's layout, config dirs and model registry) and the engine set from `shuttleengines.NewWithSeams(c.cfg, c.configDirs, LaunchSeams{RunHook: shuttleparity.UnrunHooks, SandboxProbe: shuttleparity.ConfiguredSandbox})`, then calls `Report`.
`lym shuttle manifest <seat>` (audience operator, one required positional argument) calls the field and prints its answer.
- With nothing missing it prints `output.Ok` with both manifests and the parity verdict, `Missing`, plugin contexts listed unrun, and exits zero.
  The printed verdict is `Missing` alone: each record's stored `Verdict` lists every plugin context as omitted, since `UnrunHooks` runs none, so only a real launch's record verdict says whether a context was delivered.
- With an engine that cannot start the seat, the `output.Ok` report names that engine on a line of its own with its unmet requirements from `shuttleparity.Unmet`, and the exit stays zero when nothing is missing.
  Every seat is compared, a seat Codex cannot start included, so an unmet requirement is reported and never fails the verb.
- With an item missing it prints `output.Err` naming each missing item as `<channel> <key>`, with the report, and exits non-zero, with the way forward: give the Codex engine a delivery for each missing item in `internal/shuttleengine/codexengine`, and run no Codex bench of this seat until the verb passes.
  A missing compared item is the verb's only non-zero exit besides an error from `Report`, as D10 says.
`Long` states the bounds: it starts no session and runs no plugin hook or sandbox probe;
it writes nothing outside its temporary directory but the Codex home a Codex `Prepare` readies (its links, trust record and `config.toml`), and keeps no manifest copy, since it never calls `Runner.launch`;
plugin contexts are listed by plugin and hook unrun, and only a run's record says whether one was delivered;
the developer-instructions size check leaves out plugin contexts, so a seat it passes can still be refused at a real launch.
`Command` adds `manifestCmd`, and its doc comment names the verb; the verb takes the strict cwd gate of every other verb.
`shuttleparity`'s `doc.go` names `Report`, and `docs/overview.md` adds the verb to its shuttle verb list and `internal/shuttleparity` to its package tree.

The entries `internal/seatengine.Prompts`, `internal/shuttleengine.MissingItems` (card 2), `internal/shuttleengines.NewWithSeams` (card 5), and `internal/seatengine.Specs`, `internal/landingshed.FixReviewSeatTable` and `internal/burlerengine.Engine.SeatTables` (card 6) of `testOnlyCodeAllowed` in `cmd/lym/testonlycode_linux_test.go` are deleted.
`shuttlecli` now reaches each of them: `NewWithSeams` from the production closure, `MissingItems` through `Report` and `Compare`, and the rest through `shuttleparity.Report`, `seatsOf`, the catalog's `Build` functions and `seatengine.Specs`, so every entry would match no unreachable function and the stale-entry check of `PATTERN-test-only-code` would fail.

Tests, tier0:
- `TestReport` calls `report` over a fake engine set of two of card 6's `manifestEngine` fakes on a catalog of fixture `seatTable` entries: matching manifests give no `Missing`; a Codex manifest lacking a compared item gives exactly that item; a Codex plugin context omitted as unrun gives none;
  a Codex fake meeting no requirement gives `Unmet` for a forks seat; an unknown seat name is refused naming the seats;
  with a second fixture entry whose `Build` fails, the first entry's seat still reports, and the failing entry's seat name is refused naming that builder and its error;
  after each row the temporary directory is gone, so no `manifests` copy survives anywhere.
- `TestManifestCmd` builds `shuttleCLI` by hand, as `TestOpenCmd_ListModels` builds its receiver, with a `manifestReport` stub returning a hand-built `ManifestReport` or error per seat name, and runs `manifestCmd`:
  a matching seat prints `ok` with both manifests and exits zero;
  a seat with a missing Codex item prints the error naming `<channel> <key>` and exits non-zero;
  a seat with an unmet Codex requirement and nothing missing prints `ok` naming `codex` and the requirement and exits zero;
  the stub's map keys and the expected engine name come from card 6's exported engine-name constants of `internal/shuttleparity`, never a literal;
  an error from the stub prints it and exits non-zero.
- `TestCommand_PositionalArgValidation` gains a row: `lym shuttle manifest` with no argument is refused by the positional check.

**ImpactSummary:** One new verb in shuttle's subtree; `shuttleCLI` gains one field and `Command` one `AddCommand` entry; no existing verb changes; `testOnlyCodeAllowed` loses its `internal/seatengine.Prompts` entry and the interim entries cards 2, 5 and 6 added.

**Commit:** 8: Add lym shuttle manifest
