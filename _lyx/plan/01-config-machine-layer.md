# Card 1 — config-machine-layer

**Create:**
- `plan:internal/configengine#LayerMachine` -> `const LayerMachine Layer = "machine"`
- `plan:internal/configengine#CategoryMachine` -> `const CategoryMachine Category = "machine"`
- `plan:internal/configengine#MachineFile` -> `func MachineFile(board, module string) string`
- `plan:internal/configengine#MachineKeys` -> `func MachineKeys(spec Spec) []string`
- `plan:internal/configengine#ValidateMachine` -> `func ValidateMachine(spec Spec, doc []byte) error`
- `plan:internal/configengine#readLayers` -> `func readLayers(dirs Dirs, spec Spec, anchorLayers bool) ([]LayerDoc, error)`
- `plan:internal/configengine#stackLayers` -> `func stackLayers(dirs Dirs, spec Spec, anchorLayers bool) (Stack, error)`

**Edit:**
- `internal/configengine#LayerFile`
- `internal/configengine#Layers`
- `internal/configengine#StackLayers`
- `internal/configengine#layerRank`
- `internal/configengine#load`
- `internal/configengine#Provenance`
- `internal/configengine#layerPath`
- `internal/configengine#Spec`
- `internal/configengine#Set`
- `internal/configengine#warnUnknownKeys`
- `internal/configengine#setSparse`
- `internal/configengine#setEntries`
- `internal/configengine#Edit`
- `internal/configengine/doc.go`
- `internal/configengine_test#layerSpec`
- `internal/configengine_test#newLayerFixture`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine_test#TestValidateLocal`
- `internal/configengine_test#TestLayers`
- `internal/configengine_test#TestLayerFile`
- `internal/configengine_test#TestProvenance`
- `internal/configengine_test#TestSet`
- `internal/configengine_test#TestEdit`
- `PATTERN.md`
- `pattern/PATTERN-config-layers.md`
- `pattern/PATTERN-hub-wide-config.md`
- `pattern/PATTERN-durable-vs-ephemeral-state.md`

**Uses:**
- `internal/configengine#ScratchDir`
- `internal/configengine#ValidateLocal`
- `internal/configengine#LocalKeys`

**Intent:**
Add the `machine` layer and the `machine` key category to `internal/configengine` (decision record D1, D2, D5).

`LayerMachine` ("machine") names the layer; `CategoryMachine` ("machine") is a key whose value is a property of the machine, bounding a resource every worktree of the hub on that machine shares.
`MachineFile(board, module)` returns `<board>/.lyx/config/<module>.machine.yaml`, joined through `ScratchDir(board)` so `.lyx` is never spelled (`PATTERN-lyxdirs-single-declarer`); it sits beside `LocalFile` and no other package forms the path.
`MachineKeys(spec)` returns the sorted `Categories` paths classified `CategoryMachine`, beside `LocalKeys` and `LocalDenied`, through `categoryPaths`.

`LayerFile(dirs, spec, LayerMachine)` returns `MachineFile(dirs.Board, spec.Name)` and true only when `spec.HubWide`, `MachineKeys(spec)` is non-empty, and both `dirs.Board` and `dirs.Anchor` are set.
So a `Dirs{Board}` load, which every hub-file check makes, stacks no machine layer; a standalone load (`Board` empty) never reads one; and a stray machine file of a module with no `machine` key, such as `fabric.machine.yaml`, is never read.

The stack is template, default, worktree, machine, local: `Layers` reads them in that order, and `layerRank` ranks template 0, default 1, worktree 2, machine 3, local 4, so `BelowLayer(..., LayerLocal)` includes the machine file and `BelowLayer(..., LayerMachine)` does not.
A machine file whose units are empty (empty or comment-only) is dropped as an empty local file is; a present one is checked by `ValidateMachine`, and a failure is an `ErrInvalid` error `machine config file <path>: <err>`, naming the file and the key.

`ValidateMachine(spec, doc)` has `ValidateLocal`'s shape: the same `LeafPaths` units, template-existence and shape checks, and a retired key judged by its recorded category alone.
It admits a unit only when the longest covering `Categories` entry (or the covering retired key's category) is `CategoryMachine`.
Every refusal names the key and a way forward that progresses from a machine file holding it, and is marked `ErrInvalid`:
- a covered unit of any other category: `key %q is <category> and cannot be set in a machine override; way forward: remove it from the machine override and set it in the hub file through "lyx config <module>" from the prime`;
- a unit a retired key of another category covers: `key %q is retired and cannot be set in a machine override; way forward: remove it from the machine override`, since the hub file cannot take a retired key either;
- an uncovered unit: `key %q is not classified for a machine override; way forward: remove it from the machine override`;
- a unit the template lacks: `key %q is not in the <module> template; way forward: remove it from the machine override`;
- a unit whose shape differs from the template's, and an unparseable document, as `ValidateLocal` words them, with the same removal way forward.
`Layers` wraps each with the machine file's path, so the message names the file the removal applies to.
`ValidateLocal` is unchanged: a `CategoryMachine` key is not `CategoryNone`, so the local layer keeps refusing it.
A retired key recorded `CategoryMachine` is admitted by `ValidateMachine` and dropped from the stacked document by the existing `dropRetired`, with no warning; `warnUnknownKeys` keeps skipping every layer but default and worktree.

The Anchor-and-Board rule is judged on the told `Dirs` (decision record note):
`load` and `Provenance` today clear `Anchor` for an anchor with no `_lyx/` before stacking, which would drop the machine layer.
Move the reader into `readLayers(dirs, spec, anchorLayers)`, where `anchorLayers` false skips only the worktree and local layers, and the merge into `stackLayers(dirs, spec, anchorLayers)`;
`Layers` and `StackLayers` become `readLayers(dirs, spec, true)` and `stackLayers(dirs, spec, true)`, and `load` and `Provenance` pass the told `dirs` with `anchorLayers` false where they cleared `Anchor` before.
`load` keeps judging `hasBase`, `missingBaseError` and the fill on the same geometry as today.

`layerPath` returns `dirs.Board` as the owner of `LayerMachine`, so `Set` and `Edit` check the board dir's `_lyx/`.
`setSparse` and `setEntries` check a `LayerMachine` document with `ValidateMachine` before writing, as they check a local one with `ValidateLocal`, so a refused key writes nothing.
`Edit` treats `LayerMachine` as it treats `LayerLocal`: an absent file is scaffolded empty, and a saved document `ValidateMachine` refuses is put back to its previous bytes, or removed when the call created it.

The comments the change makes stale name the machine layer too: `Spec`'s `RetiredKeys` field comment (the machine layer admits a retired key by its recorded category as the local layer does), `Set`'s doc comment and `set.go`'s header (a machine write is checked by `ValidateMachine` before anything is written, as a local one is by `ValidateLocal`), and `warnUnknownKeys`'s comment (the machine layer is skipped like the local one, since it refuses a unit the template lacks).
The doc comment of `Layer`'s constant block, `Category`'s, `LayerFile`'s, `Layers`'s and the package doc's `# Layers` and `# The local rule` sections name the machine layer: its file, its stack position, its Anchor-and-Board applicability, its open set (`MachineKeys`) and its rule (`ValidateMachine`), and that no reconcile reads or writes it.

PATTERN docs, in the same commit (`## PATTERN changes`):
`PATTERN.md`'s two index lines become, verbatim:

```
- `PATTERN-hub-wide-config` — A hub-level config module: `HubWide` in `configreg`, its tracked file on the board only, its machine override never-tracked beside it. — [background](pattern/PATTERN-hub-wide-config.md)
- `PATTERN-config-layers` — Resolving a module's config: only `configengine` reads, stacks and locates layer files, the hub-wide machine layer included. — [background](pattern/PATTERN-config-layers.md)
```

`pattern/PATTERN-config-layers.md` gains **Machine** in its category list (a key whose value is a property of the machine; it opens the key to the machine layer and keeps it closed to the local layer), and a `## Machine layer` section: the open set is `MachineKeys`, the layer applies only for a hub-wide module with a `machine` key when the told `Dirs` carry both `Board` and `Anchor`, a hub-file check loads `Dirs{Board}` and so never stacks it, and no reconcile, `hubreconcile.Ensure` included, reads, rewrites or removes the file; its `## Local files` removal list is unchanged.
`pattern/PATTERN-hub-wide-config.md` gains a bullet: the module's never-tracked machine override is `<BoardDir>/.lyx/config/<module>.machine.yaml`, holding only its `machine` keys, never in a worktree.
`pattern/PATTERN-durable-vs-ephemeral-state.md`'s background gains, beside the `.local.yaml` line, the machine file's path under the board dir's `.lyx/config/`; its index entry is unchanged.

Tests, each a row or field of the test named:
- `layerSpec` stays per-worktree; the hub-wide machine rows set `HubWide` and reclassify `port` as `CategoryMachine` through a new row field.
- `newLayerFixture` writes `LayerMachine` files at `MachineFile(fx.board, layerModule)` and gains a `geometryBoardOnly` geometry that tells `Dirs{Board}` alone.
- `TestLayerFile`: the machine file of a hub-wide spec with a machine key under `Dirs{Anchor, Board}` is `filepath.Join("/b", lyxdirs.DotLyxDirName, "config", "mod.machine.yaml")`; it does not apply to a per-worktree spec, a hub-wide spec with no machine key, `Dirs{Board}` alone or `Dirs{Anchor}` alone; `MachineKeys` of the machine spec is `[port]` and of `layerSpec()` empty.
- `TestLoad_LayerStack`: a machine `port: 8` over a default `port: 1` resolves 8; the same with `geometryAnchorWithoutLyx` resolves 8; under `geometryBoardOnly` and `geometryStandalone` it resolves the default's (or worktree's) 1; a hub-wide spec with no machine key ignores a machine file holding `name: m` and `port: 8`; a comment-only machine file over a default `port: 1` resolves 1; a retired key recorded `CategoryMachine` in the machine file loads and is absent from the result (`wantAbsent`), with no warning assertion of its own, since `warnUnknownKeys` never reads the machine layer.
- `TestValidateLocal` gains a `validator` field, `local` by default, so every existing row keeps running `ValidateLocal` with its existing verdict; new rows set `machine` and run `ValidateMachine` over a hub-wide spec classifying `port` `CategoryMachine`, `name` `CategoryNone` and `run` `CategoryExecutable`, retiring `gone_machine` as `CategoryMachine` and `gone_guard` as `CategoryGuard`:
  a machine key passes; an empty document passes; `name` (`CategoryNone`) is refused naming `name`; `run` is refused naming `run` and `lyx config mod`; a key the template lacks, an unclassified key (`list`), a mapping where the template holds a scalar (`port:\n  a: 1\n`) and an unparseable document are refused naming the key (or `parse`) and `remove it from the machine override`; `gone_machine` passes and `gone_guard` is refused naming `retired`.
  One `local` row over the same spec pins that `ValidateLocal` refuses `port`.
- `TestLayers`: the hub-wide machine spec over all four files stacks default, machine, local; with a comment-only machine file it stacks default and local; a machine file holding a non-machine key fails naming the machine file and the key, marked `ErrInvalid`.
  The `BelowLayer` subtest gains the machine spec over all four files: `BelowLayer(..., LayerLocal)` holds the machine file's `port` and `BelowLayer(..., LayerMachine)` holds the default's.
- `TestProvenance` gains a geometry and spec field: a machine-held unit is annotated `machine`, also under `geometryAnchorWithoutLyx`.
- `TestSet` and `TestEdit` build their spec with no `HubWide` and their board dir with no `_lyx/` (`set_test.go` and `edit_test.go` fixtures), and `LayerMachine` applies only to a hub-wide spec with a machine key and checks the board dir's `_lyx/`; so each gains a `hubWide` row field that marks the spec hub-wide, and its machine rows classify the key `CategoryMachine` and create `ConfigDir(board)`.
- `TestSet`: a `LayerMachine` write of the machine key creates the file holding only that key; a write of a non-machine key is refused naming it and leaves no file; a `LayerMachine` write under `Dirs{Board}` alone is refused as `does not apply here` and leaves no file; a `JSONEntries` hub-wide spec with a machine key refuses a `LayerMachine` entry of a non-machine key and leaves no file, reaching `setEntries`'s check.
- `TestEdit`: a machine save of a non-machine key is refused and the file is put back to its previous bytes, or removed when the edit created it.

**ImpactSummary:** Additive inside `configengine`: the machine layer applies only to a hub-wide spec with a `machine` key, and no registered module has one until card 2, so every existing load, set and edit resolves as before; `Layers`/`StackLayers` keep their signatures for their external callers (`modelspec`, `runprofile`, `reviewbudget`, `batcher`, `configsync`).
