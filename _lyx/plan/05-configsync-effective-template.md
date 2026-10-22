# Card 5 — configsync-effective-template

**Edit:**
- `internal/configsync#ReconcileAll`
- `internal/configsync#reconcileBaseWorktreeFile`
- `internal/configsync#reconcileSparseFile` -> `func reconcileSparseFile(m configreg.Module, dirs configengine.Dirs, path string, layer configengine.Layer, apply bool, strip *migration) (result Result, present bool, err error)`
- `internal/configsync#stripToSparse`
- `internal/configsync#stripBase` -> `func stripBase(m configreg.Module, base, current []byte) ([]byte, []string, error)`
- `internal/configsync#sparseSeed`
- `internal/configsync#stripMovedUnits`
- `internal/configsync#moveKeys`
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#moduleWant`
- `internal/configsync#TestReconcileAll`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`
- `internal/configsync#TestReconcileAll_MigratesToSparse`
- `internal/configsync#TestReconcile_FailureNamesItsFile`
- `internal/configsync#TestReconcile_MovesKeysAcrossModules`
- `internal/configcli/doc.go`

**Uses:**
- `plan:internal/configengine#EffectiveTemplate`
- `plan:internal/configengine#TemplateUnder`
- `internal/configengine#LayerFile`
- `internal/configengine#BaseLayer`
- `internal/configsync#writeModuleFile`
- `internal/configsync#findResult`

**Intent:**
Judge every configsync strip, prune, reconcile and template-equal drop against the effective template, so a reconcile drop leaves the loaded value unchanged under any `lens_set` (D4).
Paths that list template keys or scaffold a whole template keep the base template: the seed-only branches, the key-move scaffold and `templateValueNode` in `movekeys.go`, and `retireHubWideCopy`, which judges through `BelowLayer`.

`reconcileBaseWorktreeFile` composes, after the Migrate hook, `configengine.EffectiveTemplate(m.Spec(), existing, false)` from the standalone file it reconciles, and passes it to `yamlengine.Reconcile`; a seed-only module keeps `m.Template()`.
So under a standalone `lens_set: csharp`, a lens entry equal to the csharp prose is dropped and one equal to the base prose is kept.
`reconcileSparseFile` takes `dirs` and prunes against `configengine.TemplateUnder(dirs, m.Spec())`; `ReconcileAll` passes `configengine.Dirs{Anchor: baseDir, Board: boardDir}` for the worktree and local files.
`stripToSparse` judges the repo default's differing keys against `EffectiveTemplate(spec, repoDefault, true)` instead of `spec.Template`; its strip of the worktree file already goes through `BelowLayer`.
`stripBase` takes `base`, the base document whose selector picks the template, and strips `current` against `EffectiveTemplate(spec, base, true)`;
`ReconcileHubWideAt` passes the hub file it reconciles as both, `sparseSeed` passes its migrated seed as both, and `stripMovedUnits` passes the whole file as `base` and the picked moved units as `current`, so a moved unit is judged under the file's own `lens_set`.
`sparseSeed` prunes and strips against `EffectiveTemplate(spec, <migrated seed>, true)`.
`ReconcileHubWideAt` prunes a present non-seed-only file, and runs `DropTemplateEqualNames`, against `EffectiveTemplate(spec, mergedBytes, true)` of the migrated file.
An unknown selector value fails the module's reconcile with a `*FileError` naming the file that holds the value and carrying `configengine`'s unknown-value error.
Where the reconciled file is the base document the template is composed from, that is the reconciled file.
On every path that composes the template from another file, the error names that file:
on `reconcileSparseFile`, whose prune and strip of a hub worktree or local file judge against the base layer, it names the base layer's file, `configengine.LayerFile` at `configengine.BaseLayer`;
on the `SeedPrime` path and on `moveKeys`' `loadTarget` seed through `sparseSeed`, where the value sits in the prime's copy, it names that copy, `configengine.ConfigFile(primeBaseDir, m.Name)` in `ReconcileHubWideAt` and `configengine.ConfigFile(seedDir, module)` in `moveKeys`.
On both prime paths `configsync` appends to `configengine`'s error, whose way forward sets the base, the step that way forward lacks: `; also delete the lens_set line from <prime copy> in an editor`, since a prime worktree file holding a selector the base does not hold is refused at load.

The header comment of `configsync.go`, the package's reconcile doc, which no member glyph spans, says every strip, prune, reconcile and template-equal drop judges a file against the effective template its base layer picks, and that key-move scaffolds and the seed-only paths use the base template.
`internal/configcli/doc.go`'s reconcile paragraph says the same of `lyx config reconcile`: it judges a file against the template its base layer's picked overlay composes.

Tests.
`moduleWant` gains `loadUnchanged`, which makes `TestReconcileAll` load `lenses.Load` over the base dir before the run and assert the load after the run equals it.
`TestReconcileAll` gains an apply row: a standalone `lenses.yaml` holding `lens_set: csharp`, `concurrency` with the csharp prose and `error-handling` with the base prose wants `droppedAll` `lenses.concurrency`, a file still holding `error-handling:` and `lens_set: csharp`, and `loadUnchanged`; the csharp prose is read by loading `lens_set: csharp` into a scratch anchor.
`TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` gains a `loadUnchanged` row field, comparing `lenses.Load(configengine.Dirs{Board: boardDir})` before the dry run and after the apply, and two `lenses` rows:
a present hub file of the same shape wants `SeedHub`, `wantDropped` `lenses.concurrency`, `wantNotDropped` `lenses.error-handling` and `loadUnchanged`;
an absent default seeded from a prime file of the same shape under `seedDefaults` wants `SeedPrime`, `wantContains` `lens_set: csharp` and `error-handling:`, and `wantAbsent` `concurrency:`.
`TestReconcileAll_MigratesToSparse` gains a `lenses` row: a repo default `lens_set: csharp\n` under a worktree file holding `concurrency` with the csharp prose wants `wantStripped` `lenses.concurrency`, `wantInherited` `lens_set` (the default's value differs from the template's `""` and the worktree file lacks it), the worktree file deleted and the marker written.
`TestReconcile_FailureNamesItsFile` gains a `content` row field, the file's bytes, empty keeping `unparseable`, a `wantErr` field of substrings the message must hold, a `worktreeFile` field, bytes written as the module's file in the base dir beside a `content` file at the board dir, and a `burlerFile` field, bytes written as the board's `burler.yaml`, and five `lenses` rows with `content` `lens_set: nope\n` and `wantErr` holding `names no shipped layer`:
`ReconcileAll` standalone (`boardDir` passed empty) with the file in the base dir;
`ReconcileAll` in a hub with the file at the board dir and `worktreeFile` `fans:\n  mine:\n    - generic\n`, wanting the board file's path;
`ReconcileHubWideAt` with the file at the board dir;
and `ReconcileHubWideAt(boardDir, baseDir, HubWideOptions{SeedDefaults: true})` with no board file and the file in the base dir as the prime's copy, wanting the prime's path and `wantErr` also holding `delete the lens_set line from`;
and the same call with `burlerFile` `fans:\n  theirs:\n    - generic\n`, so `moveKeys`' `loadTarget` seeds the board `lenses.yaml` from the prime's copy first, wanting the prime's path and `delete the lens_set line from`.
`TestReconcile_MovesKeysAcrossModules` gains a `stripped` row field, checked only when set: the lenses `Result`'s `Stripped` holds each entry.
The `board` call runs without `StripBase`, so the strip's bytes are only reported and the file's `concurrency` is dropped by `DropTemplateEqualNames` either way; `stripped` is the assertion that pins `stripMovedUnits`' judgment under the file's own `lens_set`.
It gains a `board` row: a prime `lenses.yaml` holding `lens_set: csharp\n` and a board `burler.yaml`, literal YAML holding only a `lenses` mapping so the move leaves it empty, whose `lenses` holds `concurrency` with the csharp prose (read by loading `lens_set: csharp` into a scratch anchor) and a `hub-lens`, so the move creates the board `lenses.yaml` from the prime's seed and `stripMovedUnits` judges the moved lenses under the file's own `lens_set`: `contains` `lens_set: csharp` and `hub-lens:`, `excludes` `concurrency:`, `stripped` `lenses.concurrency`, `moved` `burler.lenses -> lenses.lenses`.
No row pins `reconcileSparseFile`'s prune result under the effective template, beside its failure row above: a prune never removes a per-name name, and an overlay adds only per-name names, so its result is the same under either template.

**ImpactSummary:** every changed path calls the same yamlengine operation with a template that equals `m.Template()` for every module but lenses, and for lenses whenever `lens_set` is empty; `reconcileSparseFile` and `stripBase` are re-signed with all their callers on this card, and `movekeys.go`'s `sparseSeed` call keeps its signature.
