# Card 20 — batten-switch-entry-gate

**Create:**
- `internal/battencli#stepGatedOffCommandRow`

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
- `internal/shedrecipe#commandRefWayForward`
- `internal/shedrecipe#commandTestConfig`
- `internal/shedrecipe#commandTestSettings`
- `internal/shedadapters#CommandProducer`
- `internal/battenshed#Config`
- `internal/battenshed#ConfigSpec`

**Intent:**
Batten's Open-Window, Seed-IDE and Open-IDE rows move their `switch:` to `entry_gate:` (D10), so the Command producer reads no switch.

The recipe's three rows carry `entry_gate: batten.open_window`, `batten.seed_ide` and `batten.open_ide` and keep `command:`; its header comment says a row whose setting is off is skipped by its entry gate.
`commandEntry` accepts `command` and `timeout_s` only, so a leftover `switch` key is refused as unknown, and `commandKeyWayForward` becomes "way forward: remove the key; a Command row takes command and timeout_s, and a switch moves to the row's entry_gate"; `commandRefWayForward` loses the word switch and tells the writer to set command to a module.key setting reference in the row's config.
`CommandRow` loses `SwitchModule` and `SwitchKey`, and `CommandDeps` loses `Forced`.
`CommandProducer.Call` runs its command whenever it is called: the switch read, its non-bool refusal and the forced branch go, the non-bool refusal surviving as card 19's evaluation error.
`battenCLI.commandDeps` sets no `Forced`.

`lyx batten run --window` opens a window running `lyx batten run --open-terminal`, and for that inner run `battenCLI.wire`'s `Env.Setting` answers `batten.open_window` true, whatever the task's config says; every other lookup is unchanged.
Bound: the hidden `--open-terminal` flag forces only Open-Window's own setting for its own run.
`battenCLI.reopenWindow` reads `batten.open_window` through `Env.Setting` itself and opens nothing when it is off, so a forced run and a task's `open_window: false` apply as in the row.
`internal/battenshed/template.yaml`'s comments say each setting is the row's entry gate.

`TestCommandProducer` drops its switch rows and keeps the command rows.
`TestCommandEntry` updates the expected text of its `UnknownKey` row to the new way forward, which a leftover `switch` key reaches by the same `configRejectUnknown` path, updates its local copy of the reference way forward to `commandRefWayForward`'s new text, deletes its `MissingSwitch` and `NonStringSwitch` rows, and points its `ReferenceWithNoDot` row at `command` (`open_window_command`, wanting `command` and the way forward), since that row is the only one reaching the reference's dot check, which `command` keeps.
The test helper config `commandTestConfig` drops the `switch` key, so the conformance table's Command row, built from it, still builds, and `commandTestSettings`'s comment no longer names the switch.
`TestBattenPreRun_ReopensTheWindowWithNoClientAttached` keeps its `SwitchOff` row, now answered through `Env.Setting`, and deletes its `SwitchOffAndOpenTerminalForced` row with the `forced` field: its fake `Setting` answers one plain bool, so a rewritten row would equal `PastOpenWindowWithNoClient`.
The other users of the deleted `Forced` move too: `TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` asserts that `Env.Setting` answers `batten.open_window` true under `--open-terminal` and refuses without it, in place of `c.env.Command.Forced`, the one place the wired forced answer is pinned; `stepCommandRows_FollowTheTasksLocalSwitch` keeps its marker assertions, which prove the forced Open-Window run through the entry gate.
Its off-case steps, Open-IDE with `open_ide` off and Open-Window with `open_window` off, go through `stepGatedOffCommandRow`, since a gated-off row now ends its step skipped and chains on, which `stepCommandRow`'s Done check refuses: the helper seeds a running status at the row with `pause_before` naming the row's `on_done` row, steps once, and fails unless the history ends in a `skipped` entry for the row and the run is paused at that `on_done` row, so no later row runs.
For the forced step the test re-wires its CLI through `battenCLI.wire` with `openTerminalFlag` set, in place of setting `Forced`, since the forced answer is built at wiring, and re-applies the fixture's seams before stepping Open-Window.
The readers of the `switch` key, found by a search for it over the module's tests and docs, move too: shedbuild's `engineMinimalConfig` drops the `Command` entry's `switch` key, and the Command rows paragraphs of `internal/battenshed/doc.go` and `docs/overview.md` say each row runs only when its entry gate's `batten.yaml` setting is on and that a resume reopens the window when `batten.open_window` is on, in place of the switch.
`TestNew_OnDoneChainAndLoadBearingEdges` asserts the built Open-Window, Seed-IDE and Open-IDE rows carry entry gates that answer false when a fake `Setting` returns false for their setting and true when it returns true.
The shedadapters Command section and the battencli package doc in `cli.go`, with the run verb's long help, describe the entry gate in place of the switch.

**ImpactSummary:** Moves batten's three Command switches into entry gates and removes the producer's switch and force; touches the recipe, the Command producer and entry, batten's wiring, shedbuild's engine fixture and the docs that describe the switch.
