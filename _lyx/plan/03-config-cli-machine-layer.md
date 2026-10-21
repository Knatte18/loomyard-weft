# Card 3 — config-cli-machine-layer

**Create:**
- `plan:internal/configcli#setMachine` -> `func setMachine(dirs configDirs, out io.Writer, mod configreg.Module, path string, pairs []yamlengine.KV) int`
- `plan:internal/configcli#editMachine` -> `func editMachine(dirs configDirs, out io.Writer, mod configreg.Module, path string, edit configengine.EditorFunc) int`

**Edit:**
- `internal/configcli#layerNames`
- `internal/configcli#validLayer`
- `internal/configcli#configDirs`
- `internal/configcli#dispatch`
- `internal/configcli#noTerminalRefusal` -> `func noTerminalRefusal(module, layer string) string`
- `internal/configcli#menu`
- `internal/configcli#configDirs.selectedLayer`
- `internal/configcli#editOne`
- `internal/configcli#setModule`
- `internal/configcli#buildConfigLong`
- `internal/configcli#Command`
- `internal/configcli/doc.go`
- `docs/overview.md`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#runMenuWith`
- `internal/configcli#TestMenu`
- `internal/configcli#TestConfigOverRealHub`

**Uses:**
- `plan:internal/configengine#LayerMachine`
- `plan:internal/configengine#MachineFile`
- `plan:internal/configengine#MachineKeys`
- `internal/configcli#setChecked`
- `internal/configcli#newLoadFailure`
- `internal/configcli#rollBack`
- `internal/configcli#snapshotFile`
- `internal/configcli#configDirs.refuseHubWrite`
- `internal/configcli#configDirs.layersHolding`
- `internal/configcli#hubWriteRefusal`
- `internal/configcli#newHubFixture`
- `internal/configcli#fakeEditor`
- `internal/configcli#assertJSONErrContains`
- `internal/configcli#boardHead`

**Intent:**
Let `lyx config` address the machine layer (decision record D4, D5).

`layerNames` gains `configengine.LayerMachine`, ordered default, worktree, machine, local, so `validLayer` takes `--layer machine` and the menu's `layersHolding` marks a module whose machine file exists with `machine`, as `gate (default, machine)`.
`dispatch`'s unknown-layer refusal becomes `unknown --layer %q; use default, worktree, machine or local`, and the comments of `validLayer` and of `configDirs`' `layer` field, which say "one of the three layer names", name the four.
`buildConfigLong` gains only the `machine` token in its two lists, as `--layer default|worktree|machine|local` and `layer: template, default, worktree, machine or local`, since its Long text measures about 1978 of the 2000 bytes `cmd/lyx/clitree_test.go`'s `maxLongBytes` allows.
The gloss, a never-tracked override under the board dir's `.lyx`, written from the prime, of a hub-wide module's `machine` keys, goes into the `--layer` flag help in `Command`, whose list becomes `default, worktree, machine or local`, and into `configcli/doc.go`, never into Long.

`selectedLayer` refuses `--layer machine`:
on a module that is not hub-wide with `%s is a per-worktree module and has no machine file; way forward: --layer local`, reached by `--layer machine` on `loom`;
on a hub-wide module whose `configengine.MachineKeys` is empty with `%s has no machine key; way forward: "lyx config %s" from the hub's prime worktree`, reached by `--layer machine` on `fabric`.
Both precede the guard and touch no file.

`editOne` and `setModule` route `LayerMachine` through `dirs.refuseHubWrite` first, the same guard as a default-layer write: refused from any worktree that is not the hub's prime and from a session whose strand name carries a slug, leaving the file unchanged and opening no editor.
Bound: as for the default layer, the guard stops a mistaken write by a pair or slugged session, not a direct file write; a machine file never reaches Bolt or any record, and its values still pass gate's loader at every acquire.
`setMachine` writes through `setChecked(dirs.engineDirs(), mod, configengine.LayerMachine, path, pairs)`: a key the machine layer refuses is refused by `configengine.Set` before anything is written, naming the key and the hub file's `lyx config <module>`, and a value the module's loader refuses over the prime's worktree stack, which includes the machine layer, is refused and the file put back.
It commits nothing and runs no fabric sync, and reports `edited machine override <path>; it is not committed`.
`editMachine` snapshots the file, takes `CheckLoad` over `dirs.engineDirs()`, runs `configengine.Edit(..., LayerMachine, edit)`, which puts back a save the machine rule refuses, then refuses a save after which the stack newly fails to load through `newLoadFailure` and puts the file back to its snapshot (previous bytes, or absent when the edit created it) with `rollBack`; an abort reports `aborted: <path> unchanged`.
It commits nothing and runs no fabric sync.
Both report a load-check refusal through `newLoadFailure`, whose trailing `way forward: --set the key to a value it takes` is the operative step: the gate loader's two-file text embedded ahead of it reads as the cause, and its removal step may name a file `rollBack` has already removed.

`noTerminalRefusal(module, layer)` names the `--layer` the refused call carried in both ways forward, as `"lyx config gate --layer machine --print"` and `"lyx config gate --layer machine --set key=value"`, and with an empty layer words them as today.
Without it, `lyx config gate --layer machine` run with no terminal, as from an orch's shell, would be sent to `lyx config gate --set`, which writes and commits the hub file on Bolt and reaches every machine.
`dispatch` passes its `layer`, and `menu`, which selects no layer, passes "".
`printResolved` needs no change: `configengine.Provenance` names `machine` for a machine-held unit.
Reconcile needs no change: `lyx config reconcile`, with or without `--layer default`, reads and writes only `_lyx/config/` files and local files under the worktree's anchor, never the board dir's `.lyx/config/`.

Docs: `configcli/doc.go`'s `# Layers` section names `--layer machine`: it addresses `configengine.MachineFile` of the board dir for a hub-wide module with `machine` keys, passes the default layer's write guard, never commits or syncs, refuses a key the machine layer closes before writing, checks the load and puts the file back, and is refused on a per-worktree module and on a hub-wide module with no machine key with their ways forward; `--resolved` and the menu name `machine`; reconcile never touches the machine file.
`docs/overview.md`'s config paragraph names the machine layer at `<BoardDir>/.lyx/config/<module>.machine.yaml` beside the local override and `--layer default|worktree|machine|local`.

Tests:
- `TestDispatchLayers` gains `seedMachine` (written at `configengine.MachineFile(fx.board, module)`) and an `editor` field (default `makeNeverCalledEditor`).
  Every gate row seeds the board's `gate.yaml` with the gate template (`seedBoard`): `gateslot.LoadConfig` returns `ErrConfigAbsent` before stacking any layer when the hub file is absent, so without it `CheckLoad` fails the same way before and after a write and `newLoadFailure` lets a `go_parallel: 0` through. Rows:
  the unknown-layer row's `wantErr` becomes `default, worktree, machine or local`;
  `gate --layer machine --set go_parallel=8` writes the machine file as exactly `go_parallel: 8\n`, with 0 commits and 0 syncs;
  the same write under a refusing guard, and under `hubWriteRefusal(fx.layout, taskSession)`, is refused and leaves no machine file;
  an editor edit of `gate --layer machine` under a refusing guard is refused, opens no editor and leaves no machine file;
  `--set go_parallel=0` over a machine file holding `go_parallel: 8\n` is refused naming `go_parallel`, carries `way forward: --set the key to a value it takes` after the loader's text and leaves the file as it was;
  an editor save of `go_parallel: 0\n` is refused through the load check, carries `way forward: --set the key to a value it takes` after the loader's text and leaves the seeded machine file's bytes, and over no seeded file leaves no file;
  an editor save of `busy_quit_sec: 9\n` over a seeded machine file is refused naming `busy_quit_sec` and leaves its bytes;
  an editor that fails on `--layer machine` reports `aborted: <machine file> unchanged` and leaves the seeded bytes;
  an editor save of `slots: 1\n` writes it with 0 commits and 0 syncs;
  `--print --layer machine` over a seeded machine file prints its bytes, and over none answers `not configured` naming the machine file's path;
  `--layer machine` on `loom` is refused naming `--layer local`, and on `fabric` naming `lyx config fabric`, each writing no file;
  `gate --layer machine` with no terminal is refused naming `--layer machine --set` and writes no file;
  `gate --set go_parallel=0` with no `--layer`, over a machine file holding `go_parallel: 8`, is refused, leaves the hub file as seeded, and its error names neither the machine file's path nor `--layer machine`;
  an editor save of the gate hub file (no `--layer`) holding `go_parallel: 0`, over a machine file holding `go_parallel: 8`, is refused and leaves the hub file as seeded, pinning that the hub-wide editor save checks `Dirs{Board}` alone.
- `runMenuWith` accepts a `machine` seed layer at `configengine.MachineFile`, and its doc comment lists the `machine` seed form; `TestMenu`'s `status lists every layer that holds a file` row adds the `gate:default` and `gate:machine` seeds and wants `gate (default, machine)`.
- `--resolved` naming `machine` is pinned by card 1's `TestProvenance` machine rows; `printResolved` passes `Provenance`'s output through, as `TestPrintResolved`'s existing subtest pins.
- `TestConfigOverRealHub`: from the prime with no strand name, `gate --layer machine --set go_parallel=8` exits 0, writes `configengine.MachineFile(boardDir, "gate")`, leaves `boardHead` unchanged and `_board`'s `git status --porcelain` empty; `guardRows` gains a `layer` field and a row writing `gate --layer machine --set go_parallel=8` from the task pair, refused naming the prime, its check reading the machine file instead of the hub file; the existing `reconcile --layer default --apply` subtest seeds a machine file first and asserts its bytes unchanged after the apply.

**ImpactSummary:** Confined to `internal/configcli` plus `docs/overview.md`: one new `--layer` value routed through the existing guard and load check; the changed existing text is the unknown-layer refusal, the help and the no-terminal refusal's way forward, which now carries a selected `--layer`; the unknown-layer row in `TestDispatchLayers` is updated here, and the no-terminal rows assert `--print` and `--set` substrings that keep matching.
