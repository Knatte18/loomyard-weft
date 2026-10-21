# Lens review

Every lens fork returned a report.
Duplicates across lenses are merged into the first line that carries them.

- affected-set, LOW, `planglyph/doc.go:76` and `contracts/specs/loom-plan-spec.md:477` name only `gate.yaml` as a slot-acquire failure: fixed as proposed; card 2 lists both files and gives each the machine override, and the plan verify runs `./internal/planglyph`.
- affected-set, LOW, `runMenuWith`'s doc comment lists only the default and local seed forms: fixed as proposed; card 3 updates the comment along with the new `machine` seed.
- way-forward, MEDIUM, the no-terminal refusal on `--layer machine` points at `lyx config gate --set`, which writes the hub file: fixed as proposed; card 3 re-signs `noTerminalRefusal(module, layer)` to carry the layer, lists its caller `menu`, and adds a `TestDispatchLayers` row.
- way-forward, LOW, the `ValidateMachine` refusal of a retired key pointed at the hub file, which cannot take it: fixed as proposed; card 1 gives a retired key its own refusal whose way forward is removing it from the machine override.
- way-forward + criteria-coverage, LOW, the uncovered-key and template-lacks refusals named no way forward: fixed as proposed; card 1 gives every `ValidateMachine` refusal the removal way forward, and `Layers` names the file.
- way-forward, LOW, `--layer machine --set` cannot fix a machine file holding a refused key: fixed as proposed (no concrete change was proposed); card 2's two-file message now names removal for the case `--layer machine` refuses a key the file holds, and says why.
- branch-coverage + criteria-coverage, MEDIUM, no row for `--print --layer machine`: fixed as proposed; card 3 adds a present row and an absent row.
- branch-coverage, MEDIUM, no row for a machine editor edit under a refusing guard: fixed as proposed; card 3 adds the row.
- branch-coverage, MEDIUM, no row reaches `editMachine`'s abort: fixed as proposed; card 3 adds an editor-failure row.
- branch-coverage + criteria-coverage, MEDIUM/LOW, the two-file text on the `configengine.Load` failure path was unpinned: fixed as proposed; card 2's machine `busy_quit_sec: 9` row asserts the exact two-file message.
- branch-coverage, MEDIUM, the `ConfigFiles` branch for a stat error other than absence had no reaching row: fixed differently. A row needing a stat failure was not added. The branch is dropped instead: the machine file is named only when `os.Stat` succeeds, since `configengine.Layers` already names the path when it fails to read it.
- branch-coverage + criteria-coverage, MEDIUM, no row for an empty or comment-only machine file: fixed as proposed; card 1 adds a comment-only machine row to `TestLayers` and `TestLoad_LayerStack`.
- branch-coverage, LOW, `ValidateMachine`'s shape check and parse error had no rows: fixed as proposed; card 1 adds both machine rows.
- branch-coverage, LOW, `TestValidateLocal`'s validator field did not say which rows run which validator: fixed as proposed; card 1 makes the field default to `local` for every existing row and specifies each `machine` row and its spec.
- branch-coverage, LOW, `layerRank`'s machine position had no row: fixed as proposed; card 1's `BelowLayer` subtest gains machine-spec assertions for `LayerLocal` and `LayerMachine`.
- branch-coverage, LOW, `setEntries`'s machine check was unreachable from any registered module: fixed as proposed; card 1 adds a `TestSet` row over a synthetic `JSONEntries` hub-wide spec with a machine key.
- branch-coverage, LOW, no row ran `Set` on `LayerMachine` where the layer does not apply: fixed as proposed; card 1 adds a `Dirs{Board}`-alone `TestSet` row.
- branch-coverage + criteria-coverage, LOW, an absent hub file beside a machine file stayed `ErrConfigAbsent` with no row and no stated reason: fixed as proposed; card 2 and the overview's Shared Decisions say why this case is outside D3's two-file message, and card 2 adds a `TestLoadConfig` row.
- branch-coverage, LOW, the `fabric.machine.yaml` and anchor-without-`_lyx/` criteria were shown only on synthetic specs: fixed as proposed. Card 2's configreg test asserts `MachineKeys` is empty for every module but gate. Card 2's `TestWebsterGeometry` row runs on an anchor with no `_lyx/`, and the card now says so.
- test-economy, LOW, the `TestLoadConfig` rows for machine `slots: 1` and `bogus: 1` repeated covered branches: fixed as proposed; both rows are dropped.
- test-economy, LOW, the second local file holding `go_parallel: 9` repeats the `slots: 9` refusal branch: disputed, every fix worse. The finding proposed one local file holding both keys, or `slots` alone. `decision-record.md:133` requires that the local layer refuses both `go_parallel` and `slots`. `internal/configengine#ValidateLocal` returns at the first refused path, so one file can show only one key refused. The card now states this reason.
- test-economy, LOW, `TestWebsterGeometry` repeated the two-file message and the removal-returns-hub-values check: fixed as proposed; only the machine-value-between-reads and the lease `-p=8` assertions remain.
- test-economy, LOW, `TestPrintResolved`'s machine row repeated `TestProvenance`: fixed as proposed; the row is dropped and card 3 points at card 1's `TestProvenance` rows.
- test-economy, LOW, `TestMenu` planned a new row where the existing multi-layer row fits: fixed as proposed; the seeds and the expected value go into `status lists every layer that holds a file`.
- test-economy, LOW, four machine editor rows covered two causes times two states: fixed as proposed; the rows are `go_parallel: 0` over a seeded file and over none, plus one `busy_quit_sec` row.
- test-economy, LOW, `TestDispatchLayers`' `--set busy_quit_sec=9` row repeated card 1's `TestSet` refusal: fixed as proposed; the row is dropped.
- test-economy, LOW, `TestConfigOverRealHub`'s pair refusal was planned as a new step: fixed as proposed; it is a `guardRows` row with a new `layer` field.
- guard-reach, MEDIUM, the `go_parallel: 0` machine rows reach `newLoadFailure` only with the gate hub file seeded: fixed as proposed; card 3 seeds the gate template on every gate row and says why.
- criteria-coverage + row-reachability, LOW, the `TestGateTest_Scenario` machine row did not assert the removal step: fixed as proposed; it wants `by removing`.
- criteria-coverage, LOW, no row pinned that the hub-wide editor save checks `Dirs{Board}` alone while a machine file is present: fixed as proposed; card 3 adds a hub-file editor save row of `go_parallel: 0` over a machine `go_parallel: 8`.
- row-reachability, BLOCKING, the reworded `GateSlots` wrapper names the anchor's file before the board dir exists, breaking `TestWebsterGeometry`'s board-path assertion at `internal/hubgeom/webstergeom_test.go:110`: fixed as proposed; card 2's wrapper keeps naming `configengine.ConfigFile(boardDir, "gate")` and appends only the machine file from `ConfigFiles`.
- row-reachability, MEDIUM, the `TestSet` and `TestEdit` machine rows could not reach `LayerMachine` (no `HubWide`, no board `_lyx/`): fixed as proposed; card 1 adds a `hubWide` row field and creates `ConfigDir(board)` for the machine rows.
- row-reachability, LOW, the retired-machine-key row's "no unknown-keys warning" assertion cannot fail: fixed as proposed; the row asserts only `wantAbsent`.
- forced-edits, LOW, the `configDirs.layer` and `validLayer` comments still said "three layer names": fixed as proposed; card 3 lists both members and rewords them.
- forced-edits, LOW, the comments of `Spec.RetiredKeys`, `Set`/`set.go` and `warnUnknownKeys` still described the local layer only: fixed as proposed; card 1 lists the members and rewords them.
- forced-edits, LOW, `gateslot.Config`'s comment still said "the resolved gate.yaml": fixed as proposed; card 2 lists `internal/gateslot#Config` and rewords it.
