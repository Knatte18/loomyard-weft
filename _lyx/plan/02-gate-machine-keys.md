# Card 2 — gate-machine-keys

**Create:**
- `plan:internal/gateslot#ConfigFiles` -> `func ConfigFiles(dirs configengine.Dirs) []string`
- `plan:internal/configreg#machineKeyRefusal` -> `func machineKeyRefusal(m Module) string`

**Edit:**
- `internal/gateslot#Config`
- `internal/gateslot#ConfigSpec`
- `internal/gateslot#LoadConfig`
- `internal/gateslot#withConfigWayForward` -> `func withConfigWayForward(err error, files []string, local string) error`
- `internal/gateslot/doc.go`
- `internal/gateslot#TestLoadConfig`
- `internal/hubgeom#GateSlots`
- `internal/hubgeom/doc.go`
- `internal/hubgeom#TestWebsterGeometry`
- `internal/gatecli#runTest`
- `internal/gatecli/doc.go`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/websterengine#TestRerunCardVerifies_Slotted`
- `internal/planglyph/doc.go`
- `contracts/specs/loom-plan-spec.md`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Uses:**
- `internal/configengine#LayerMachine`
- `internal/configengine#CategoryMachine`
- `internal/configengine#MachineFile`
- `internal/configengine#MachineKeys`
- `internal/configengine#LayerLocal`
- `internal/gateslot#seedGateConfig`
- `internal/gateslot#Lease.Env`

**Intent:**
Make gate's `slots` and `go_parallel` `machine` keys and name both files and both ways forward in every gate load failure (decision record D2, D3, D5).

`ConfigSpec` classifies `slots` and `go_parallel` as `configengine.CategoryMachine`; `busy_quit_sec` stays `CategoryCrossPair` and the retired `cli_wait_sec` stays recorded `CategoryCrossPair`, so a machine file holding either fails the load naming the machine file and the key.
Its doc comment says the two keys take a machine override and every key stays closed to the local layer, and `Config`'s doc comment says it is the resolved gate config of the hub file and this machine's override, no longer "the resolved gate.yaml".
`LoadConfig` and `hubgeom.GateSlots` keep their signatures; the stack they load through `fabricengine.ConfigDirs` now stacks the machine file, read at every acquire, and `Lease.Env` carries the resolved `go_parallel` as today.

`ConfigFiles(dirs)` returns the hub file, `configengine.LayerFile` of the base layer, followed by the machine file when `configengine.LayerFile(dirs, ConfigSpec(), configengine.LayerMachine)` applies and `os.Stat` of it succeeds.
A machine file that cannot be statted for another reason is not named here, since `configengine.Layers` then fails reading it with an error that names its path.
`LoadConfig` passes it, and the local override file (`configengine.LayerFile(dirs, ConfigSpec(), configengine.LayerLocal)` when that applies, the anchor holds `_lyx/`, the geometry in which the load stacks the local layer, and `os.Stat` of it succeeds, else ""), to `withConfigWayForward` on every failure but the absent hub file, which stays `ErrConfigAbsent` with its text unchanged, a machine file present or not.
That case is outside D3's two-file message: the failure is a missing strict base, not a value either file holds, so no step on the machine file progresses from it and the hub-file step `lyx fabric reconcile` is the one way forward; its existing clause already says what a session lyx refuses that verb from does.

`withConfigWayForward(err, files, local)` appends, with one file (no machine file, or a `Dirs{Board}` load):
`; fix it with "lyx config gate" from the prime; a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them`.
With the machine file too:
`; the hub file <hub>, the machine file <machine> or the file the error names holds the value: fix the hub file with "lyx config gate" from the prime, and the machine file with "lyx config gate --layer machine" from the prime, or, when that refuses a key the machine file holds, by removing <machine>; "lyx config gate --resolved" names the layer holding each value; a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them`.
With a non-empty `local`, both forms insert `; remove <local>, which takes no gate key` ahead of `; a session that is not the prime`, so a failure the local override causes, which neither config step touches, has a step that fixes it, and the role clause covers it too.
`local` is a separate argument rather than an entry of `ConfigFiles`, so `GateSlots`' wrapper below keeps naming only the hub and machine files.
The removal is named for the case `--layer machine` cannot fix: a `--set` re-validates the merged document with the refused key still in it, and the editor needs a terminal.
Both forms keep the substring `fix` … `"lyx config gate"` … `from the prime`, and the one-file form keeps `fix it with "lyx config gate" from the prime` whole, which `internal/verifytree`'s `verify_integration_test.go` asserts by substring.
The removal step is a way forward from the state the failure leaves: removing the file returns every gate run of the hub on this machine to the hub file's values (D3 bound), and the role clause keeps a pair from taking it.

`GateSlots`'s wrapper keeps naming `configengine.ConfigFile(boardDir, "gate")` as today and appends ` and <machine>` for each file `gateslot.ConfigFiles(fabricengine.ConfigDirs(l))` returns after its first, so it names the machine file when that is in the stack and the board's `gate.yaml` alone otherwise.
It never names `ConfigFiles`' first entry, because before the board dir exists `ConfigDirs` tells `Board` empty and that entry is the anchor's file, while `TestWebsterGeometry` asserts the board's path at that point; its doc comment and `hubgeom/doc.go`'s line say the limits are read from the board dir's `gate.yaml` and this machine's override.

`runTest`'s way forward after a failed limits read other than `ErrConfigAbsent` becomes `way forward: take the step it names, then re-run the same command`, since the wrapped error now names the steps for both files and the clause for a session that is not the prime; the `ErrConfigAbsent` branch is unchanged.

`internal/planglyph/doc.go`'s slot-acquire line and `contracts/specs/loom-plan-spec.md`'s matching sentence name "an absent or invalid `gate.yaml`" as what fails a slot acquire; both gain "or an invalid machine override of it".
Docs: `gateslot/doc.go`'s `# Config` section says `slots` and `go_parallel` take a machine-local override at `<BoardDir>/.lyx/config/gate.machine.yaml`, never reconciled, read at every acquire, and that `busy_quit_sec` stays hub-file only; `gatecli/doc.go`'s lines 42 and 50–51 say the limits come from the board's `gate.yaml` and this machine's override, and that a load failure names each file and its way forward.

`configreg`: a per-worktree module that classifies a key `CategoryMachine` fails registration.
`machineKeyRefusal(m)`, in `configreg_test.go`, returns a message naming the module and its machine keys when `!m.HubWide` and `configengine.MachineKeys(m.Spec())` is non-empty, and "" otherwise.
`TestModules_EveryTemplateKeyClassified` reports it as an error for every registered module, asserts `configengine.MachineKeys` is `[go_parallel slots]` for gate and empty for every other module, so no stray machine file such as `fabric.machine.yaml` is read by any registered loader, and holds a row that runs `machineKeyRefusal` on a synthetic per-worktree `Module` whose `Spec` classifies a key `CategoryMachine` and requires a non-empty refusal, so the check is seen failing.

Tests:
- `TestLoadConfig`: the existing exact `wantErr` strings for `zero slots`, `zero go_parallel` and `negative busy_quit_sec` gain the role clause.
  New rows over a board and an anchor, the machine file written at `configengine.MachineFile(board, "gate")`: hub `go_parallel: 4` with machine `go_parallel: 8` loads 8; machine `go_parallel: 0` fails with the exact two-file message naming both paths (the `decodeConfig` path); machine `busy_quit_sec: 9` fails marked `ErrInvalid` with the exact message, which names the machine file, the key and the two-file way forward (the `configengine.Load` path); machine `cli_wait_sec: 180` fails marked `ErrInvalid` naming the machine file and the key; a `Dirs{Board}` load over the same `go_parallel: 0` machine file loads the hub's 4; a machine `go_parallel: 8` beside an absent hub file is `ErrConfigAbsent`.
  The existing local-layer row gains `go_parallel: 9` beside `slots: 9` in a second local file, each refused naming the local file, the key and `remove <local>, which takes no gate key`; two files are needed because `ValidateLocal` returns at the first refused key.
- `TestWebsterGeometry`, whose fixture anchor has no `_lyx/`, so it also pins the gate-level load from such an anchor: after the hub file holds `slots: 3\ngo_parallel: 5\nbusy_quit_sec: 9\n`, writing `slots: 1\ngo_parallel: 8\n` to the machine file makes the next `GateSlots.Limits()` return `{Slots: 1, GoParallel: 8, BusyQuit: 9s}`, and a lease acquired from the pool has `Env` carrying `-p=8` in `GOFLAGS`.
- `TestGateTest_Scenario`: the `a gate.yaml value below 1 names lyx config gate` row's `wantError` becomes `fix it with "lyx config gate" from the prime`; a new row writes `go_parallel: 0` to the machine file and restores by removing it, and its error names the hub file, the machine file, `lyx config gate --layer machine`, `by removing` and `a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them`; the row struct's single `wantError` becomes a list where the new row needs several.
- `TestRerunCardVerifies_Slotted`'s exact `wantFailure` gains the role clause.

**ImpactSummary:** Changes gate's key classification and every gate load failure's text: `gateslot` (spec, loader, new `ConfigFiles`), `hubgeom.GateSlots`'s wrapper and `gatecli.runTest`'s way forward; exact-text assertions in `gateslot`, `websterengine` and `gatecli` are updated here, substring assertions in `verifytree` and `hubgeom` keep matching, and the two prose sites in `planglyph` and the plan spec that name `gate.yaml` as a slot-acquire failure gain the machine override.
