# Card 20 — batten-switch-entry-gate

**Edit:**
- `contracts/recipes/batten-recipe.yaml`
- `internal/shedrecipe#commandEntry`
- `internal/shedadapters#CommandRow`
- `internal/shedadapters#CommandDeps`
- `internal/shedadapters#CommandProducer.Call`
- `internal/battencli#battenCLI.commandDeps`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.reopenWindow`
- `internal/battenshed/template.yaml`
- `internal/shedadapters#TestCommandProducer`
- `internal/shedrecipe#TestCommandEntry`
- `internal/battencli#TestBattenPreRun_ReopensTheWindowWithNoClientAttached`
- `internal/shedrecipe#commandKeyWayForward`
- `internal/battencli#TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree`
- `internal/battencli#stepCommandRows_FollowTheTasksLocalSwitch`
- `internal/battenrecipe#TestNew_OnDoneChainAndLoadBearingEdges`
- `internal/shedadapters/doc.go`
- `internal/battencli/cli.go`
- `internal/shedbuild#engineMinimalConfig`
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Intent:**
Batten's Open-Window, Seed-IDE and Open-IDE rows move their `switch:` to `entry_gate:` (D10), so the Command producer reads no switch.

The recipe's three rows carry `entry_gate: batten.open_window`, `batten.seed_ide` and `batten.open_ide` and keep `command:`; its header comment says a row whose setting is off is skipped by its entry gate.
`commandEntry` accepts `command` and `timeout_s` only, so a leftover `switch` key is refused as unknown, and `commandKeyWayForward` becomes "way forward: remove the key; a Command row takes command and timeout_s, and a switch moves to the row's entry_gate"; `CommandRow` loses `SwitchModule` and `SwitchKey`, and `CommandDeps` loses `Forced`.
`CommandProducer.Call` runs its command whenever it is called: the switch read, its non-bool refusal and the forced branch go, the non-bool refusal surviving as card 19's evaluation error.
`battenCLI.commandDeps` sets no `Forced`.

`lyx batten run --window` opens a window running `lyx batten run --open-terminal`, and for that inner run `battenCLI.wire`'s `Env.Setting` answers `batten.open_window` true, whatever the task's config says; every other lookup is unchanged.
Bound: the hidden `--open-terminal` flag forces only Open-Window's own setting for its own run.
`battenCLI.reopenWindow` reads `batten.open_window` through `Env.Setting` itself and opens nothing when it is off, so a forced run and a task's `open_window: false` apply as in the row.
`internal/battenshed/template.yaml`'s comments say each setting is the row's entry gate.

`TestCommandProducer` drops its switch rows and keeps the command rows; `TestCommandEntry` asserts a `switch` key is refused with the new way forward's text.
`TestBattenPreRun_ReopensTheWindowWithNoClientAttached` keeps its `SwitchOff` row, now answered through `Env.Setting`, and rewrites `SwitchOffAndOpenTerminalForced` to answer `batten.open_window` true through the `Env.Setting` the forced run's wiring gives, in place of the deleted `forced` field.
The other users of the deleted `Forced` move too: `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` asserts the forced answer of `Env.Setting` in place of `c.env.Command.Forced`, and `stepCommandRows_FollowTheTasksLocalSwitch` forces Open-Window through `--open-terminal` in place of setting `Forced`, and asserts over its task worktree's `open_window: false` that `Env.Setting` answers `batten.open_window` false without `--open-terminal` and true with it, and that `seed_ide`'s answer is unchanged.
The readers of the `switch` key, found by a search for it over the module's tests and docs, move too: shedbuild's `engineMinimalConfig` drops the `Command` entry's `switch` key, and the Command rows paragraphs of `internal/battenshed/doc.go` and `docs/overview.md` say each row runs only when its entry gate's `batten.yaml` setting is on and that a resume reopens the window when `batten.open_window` is on, in place of the switch.
`TestNew_OnDoneChainAndLoadBearingEdges` asserts the built Open-Window, Seed-IDE and Open-IDE rows carry entry gates that answer false when a fake `Setting` returns false for their setting and true when it returns true.
The shedadapters Command section and the battencli package doc in `cli.go`, with the run verb's long help, describe the entry gate in place of the switch.

**ImpactSummary:** Moves batten's three Command switches into entry gates and removes the producer's switch and force; touches the recipe, the Command producer and entry, batten's wiring, shedbuild's engine fixture and the docs that describe the switch.
