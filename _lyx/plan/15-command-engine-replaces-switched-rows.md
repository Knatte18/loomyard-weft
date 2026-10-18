# Card 15 — command-engine-replaces-switched-rows

**Create:**
- `plan:internal/shedrecipe#commandEntry` -> `func commandEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestCommandEntry` -> `func TestCommandEntry(t *testing.T)`
- `plan:internal/battencli#runRowCommand` -> `func runRowCommand(ctx context.Context, dir, command, logPath string, timeout time.Duration) (shedadapters.CommandResult, error)`
- `plan:internal/battencli#battenCLI.commandDeps` -> `func (c *battenCLI) commandDeps(location *lyxcwd.Location, slug, executable string, locateTask func() (*lyxcwd.Location, error)) shedadapters.CommandDeps`

**Delete:**
- `internal/shedadapters/switched.go`
- `internal/shedadapters/switched_test.go`
- `internal/shedrecipe/entries_switched.go`
- `internal/shedrecipe/entries_switched_test.go`
- `internal/battencli#battenCLI.switchedActions`
- `internal/testkit/envkit#offSwitch`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/battenrecipe#battenRowEngines`
- `internal/battenshed#Config`
- `internal/battenshed#ConfigSpec`
- `internal/battenshed#TestLoadConfig`
- `internal/configreg#TestPrepareRunSettings`
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#Command`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#wireForHub`
- `internal/battencli#newFakeReceiver`
- `internal/battencli#shortPollBattenRecipe`
- `internal/battencli#TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`
- `internal/battencli#TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating`
- `internal/battencli#TestBattenIntegration_Rows`
- `internal/loomcli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/battenshed/template.yaml`
- `internal/battenshed/doc.go`
- `internal/shedadapters/doc.go`
- `contracts/recipes/batten-recipe.yaml`
- `PATTERN.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/shedadapters#NewCommandProducer`
- `plan:internal/shedadapters#CommandRow`
- `plan:internal/shedadapters#CommandDeps`
- `plan:internal/shedadapters#CommandResult`
- `internal/shedrecipe#settingRef`
- `internal/configreg#SettingLookup`
- `internal/verifyrun#Run`
- `internal/battencli#runDeploy`
- `internal/testkit/logcapture#Capture`

**Intent:** Replace the `OpenWindow`, `SeedIDE` and `OpenIDE` engines, their `Env` seams and `SwitchedProducer` with the one `Command` engine, and wire batten's three rows onto it with their names kept (D4).

Registry: `commandEntry` is the single `"Command"` map entry, and the three switched entries leave the map with their file (PATTERN-shed-recipe-registry).
It reads the row's config: `switch` and `command`, each a required `module.key` reference parsed as `settingRef` parses one, and optional `timeout_s`, a positive int defaulting to 30; any other key, a missing or malformed reference and a non-positive timeout are refused at build time, naming the row and the key.
It builds `shedadapters.NewCommandProducer(row.Name, CommandRow{...}, deps)` with `deps` being `env.Command` with `Setting` set to `env.Setting` and `ExecutableSetting` to `env.ExecutableSetting`; a nil `ExecutableSetting` is not refused here, so the producer's refusal falls when the row is reached.
`Env` loses `OpenWindow`, `SeedIDE` and `OpenIDE`, and gains `ExecutableSetting func(module, key string) (any, error)`, the owner's told executable-key lookup over the template and the board default layer only, and `Command shedadapters.CommandDeps`, a whole-struct passthrough of the run-wide facts and runner, following `Env.Landing`'s precedent (PATTERN-recipe-setting-lookup's declared text: a recipe value is never its own `Env` field).

Recipe: `contracts/recipes/batten-recipe.yaml`'s `Open-Window`, `Seed-IDE` and `Open-IDE` rows keep their names, which resume depends on, and take `engine: Command` with `config: {switch: batten.open_window, command: batten.open_window_command}`, `batten.seed_ide`/`batten.seed_ide_command` and `batten.open_ide`/`batten.open_ide_command`; the header comment describes the `Command` rows, and `shortPollBattenRecipe` is changed byte-for-byte alike.
`battenRowEngines` maps the three rows to `"Command"`; `minimalEntries` gains `"Command"` with a valid config and drops the three; `TestNames` lists `Command` in place of the three.

Config: `battenshed.Config` drops its three bool fields, which nothing reads once the switches go through `Env.Setting`.
`ConfigSpec` adds `open_window_command`, `seed_ide_command` and `open_ide_command` as `CategoryExecutable` (PATTERN-config-layers), so the local layer, and so a task's `run_settings`, refuses them, while the switch keys keep `CategoryNone`.
`template.yaml` adds, each with a comment naming its row, the placeholders and `lyx config set --layer default` from the prime as the way to change it: `open_window_command: "{lyx} reed window {task_dir} --title {slug}"`, `seed_ide_command: "{lyx} ide seed {slug}"` and `open_ide_command: "{lyx} ide open {slug}"`.

Batten wiring, in `wire`:
- `Env.Setting` resolves each key when the row is reached, through `configreg.SettingLookup(fabricengine.ConfigDirs(taskLocation))` over the task worktree located then, so a task's `run_settings` in its local layer switch a row per task; a task worktree that cannot be located is the lookup's error.
- `Env.ExecutableSetting` is `configreg.SettingLookup(configengine.Dirs{Board: fabricengine.ConfigDirs(location).Board})`, the layers `deploy` and darn's `verify` are read from; the task worktree's tracked and local `batten.yaml` never reach it.
- `Env.Command` comes from `commandDeps`: `Lyx` the executable `wire` already resolved, `Slug`, `TaskDir` the located task worktree's anchor path, `Dir` the prime's worktree, `LogPath` a `command-<row>.log` under `shedrun.ScratchDir(location, slug)`, `Run` `runRowCommand`, `Rerun` `lyx batten run <slug>`, and `Forced` holding `Open-Window` when `--open-terminal` is set, so `lyx batten run --window` opens the window with `open_window` off.
- `runRowCommand` lives in `internal/battencli/land.go` beside `runDeploy`, the one battencli file `cmd/lyx`'s `verifyRunAllowed` admits as a `verifyrun.Run` caller; it creates the log file and runs the command through `verifyrun.Run` under a timeout context, as `runDeploy` does, reporting a timeout in `CommandResult`.
- `battenCLI` loses `terminalOpener`; `switchedActions` goes.
`battenVerbTexts`' run text describes the rows as `Command` rows reading their switch and command keys; the `openTerminalFlag` and `openTerminalFlagName` comments on `battenCLI` and the `--window` and `--open-terminal` flag help in `Command` say the flag forces the `Open-Window` row's switch on.

Tests:
- `TestCommandEntry`, Tier 1 over `newTestEnv`: a missing `switch`, a missing `command`, a reference with no dot, `timeout_s: 0` and a non-int `timeout_s` are each refused naming the key; an env with nil `ExecutableSetting` still builds; and with `env.Command.Run` a recorder and the switch and command answering on and a command, calling the built producer hands `Run` a 30 s timeout when `timeout_s` is absent and 5 s for `timeout_s: 5`.
  A valid build and the unknown-key refusal stay pinned by `TestEveryEntry_RejectsUnknownConfigKey` over `minimalEntries`' `Command` row.
- `TestLoadConfig` asserts the three command keys' template values and that the dropped bool fields leave the switch keys loadable.
- `TestPrepareRunSettings` gains rows: run settings `{open_ide: true}` and `{open_ide: false}` are accepted, and `{open_ide_command: "x"}` is refused naming the key, the local layer's `CategoryExecutable` refusal.
- `TestBattenIntegration_Rows` gains `Command` steps, each on its own slug as its steps are, with the hub's board default batten.yaml (the `Dirs.Board` layer file) setting the three commands to `printf %s {slug} > <marker-under-t.TempDir>`-style commands that write a distinct marker per row, restored before the step returns:
  - with every switch on, stepping the three rows writes each marker;
  - a command key set differently in the task worktree's tracked `batten.yaml` is not run, the board default's command is, the marker the tracked value would write stays absent, and the trace captured with `logcapture.Capture` holds the ignored-value Warn;
  - with `open_ide` written on and then off in the task worktree's local layer, as Seed-Child writes a task's `run_settings`, `Open-IDE` writes its marker on and writes none off;
  - with `open_window: false` and `--open-terminal`, `Open-Window` still runs its command;
  - a run whose status stands at `Open-Window`, and one whose status stands at `Seed-IDE`, each written as an older binary leaves it with its commit marker naming that row, resumes at that row by name and writes that row's marker first;
  - `runRowCommand` over a `sleep 5` command with a 200 ms timeout returns `TimedOut` true well before the sleep ends, and over `exit 3` returns exit code 3.
- `wireForHub` writes the hub's board default batten.yaml with every command empty, so no test steps a row into running the test binary through `{lyx}`; `TestBattenIntegration_Rows` steps the three rows as Done with no `.vscode` assertion (card 12's `lyx ide seed` test owns it); `TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating` drops its switched-seam block and keeps its create-row assertions.
- `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` asserts `Env.Setting("batten", "open_window")` refuses an absent task worktree naming the slug, and that `--open-terminal` puts `Open-Window` in `Env.Command.Forced`.
- `newFakeReceiver` fills `Env.Setting` with a lookup answering the switches off, and `FullEnv` fills `ExecutableSetting` with `TemplateSetting` and `Command` with no-op facts and runner; `nilLegal` and loom's `intentionallyNil` drop the switched paths, the latter naming `Command`'s and `ExecutableSetting`'s as filled only by battencli.

Docs: `PATTERN.md`'s `PATTERN-recipe-setting-lookup` line becomes the declared text, and a `PATTERN-command-row` entry is added under "Shed and loom" with its declared text: "Adding a side-action row to a recipe: a `Command` row over its config keys, its command read from the template and board default layer only, never a new engine."
`internal/shedadapters/doc.go` drops the switched-producer section; `internal/battenshed/doc.go` and `docs/overview.md`'s batten paragraph describe the three `Command` rows, their keys, `lyx config set --layer default` and the ignored-value warning.

**ImpactSummary:** Three registry engines, their Env seams and one producer go; batten is their only consumer and is rewired here, loom wires none, and every test naming the old seams is listed.

**Commit:** 15: One Command engine replaces batten's switched rows
