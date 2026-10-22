# Card 2 — configengine-overlay-writes

**Create:**
- `internal/configengine#scaffoldWithoutPerNameEntries`

**Edit:**
- `internal/configengine#Set`
- `internal/configengine#setSparse` -> `func setSparse(dirs Dirs, spec Spec, layer Layer, path string, pairs []yamlengine.KV) ([]string, error)`
- `internal/configengine#Edit`
- `internal/configengine#BaseScaffold`
- `internal/configengine_test#TestSet`
- `internal/configengine_test#TestEdit`
- `internal/configengine/doc.go`
- `internal/configcli/doc.go`

**Uses:**
- `internal/configengine#Overlay`
- `internal/configengine#EffectiveTemplate`
- `internal/configengine#TemplateUnder`
- `internal/configengine#overlayTemplate`
- `internal/configengine#selectorValue`
- `internal/configengine#selectorRefusal`
- `internal/configengine_test#overlaySpec`
- `internal/configengine#scaffoldIfMissing`

**Intent:**
Make `Set` and `Edit` judge against the effective template, scaffold an overlay module's base without per-name entries, and refuse the selector in a hub worktree file (D3, D4).

`scaffoldWithoutPerNameEntries` returns `spec.Template` with every `PerNameMaps` mapping emptied to `{}`, re-marshalled; a template that fails to parse panics, a fail-loud wiring guard, since an embedded template every loader parses cannot fail.
`BaseScaffold` returns it, instead of the template, for a non-sparse base of a spec with `Overlay.Selector` set, so no scaffold pins a base-template per-name entry over the picked layer; a sparse hub base keeps `SparseBaseHeader`, and a spec without an overlay keeps today's bytes.
`Set` scaffolds an absent standalone or seed-only base with `BaseScaffold(dirs, spec)` instead of the raw template.

`Set` shadows `spec.Template` with the effective template before any write path runs.
For a base-layer write it is composed from the document being written: the selector's value is the last pair whose key is the selector when one is, else `selectorValue` of the file's present bytes, and `overlayTemplate(spec, value, dirs.Board != "")` composes it;
so `--set lens_set=csharp` into a standalone file drops a per-name entry equal to the csharp prose through `SetValues`' existing template-equal rule.
For any layer above the base it is `TemplateUnder(dirs, spec)`.
An unknown selector value fails `Set` with `overlayTemplate`'s error before anything is written, removing a file `Set` scaffolded.

`setSparse` takes `dirs` and, after `SetSparse` merges and before `validateOverride`, refuses a write to the worktree layer of a hub (`BaseLayer(dirs, spec)` is the default layer) whose merged document holds the selector with a value other than the default layer file's (`""` when that file is absent or lacks the key), a non-scalar value counting as other, the D3 refinement card 1 applies at load: it returns `selectorRefusal(dirs, spec, path, heldBefore)`, `heldBefore` being whether the file's bytes before the write held the selector, and writes nothing.
So `lyx config lenses --set lens_set=<name>` in a hub worktree whose file lacks the key and whose base picks another layer is refused naming only `--layer default` from the prime, and `configcli`'s `setChecked` returns that error unchanged, never followed by its load-failure way forward.
The refusal's input after every upstream check passes is a pair setting the selector to a value the base does not hold, or any pair into a file already holding such a value.

`Edit` on the worktree layer of a hub reads the file's previous bytes, runs `EditPath` as today, and when the saved document holds the selector with a value other than the default layer file's, puts the file back to its previous bytes, or removes it when the call created it, and returns `selectorRefusal(dirs, spec, path, previousHeld)`, as it does for a refused local save; a save without such a value is kept.

Tests.
`TestSet` gains an `overlay` row field that swaps in `overlaySpec()` renamed `testmod`, keeping the row's `layer`, `board` and `state`, and an `errLacks` field of substrings the refusal must not hold.
Rows: a standalone absent base with `pick=x` scaffolds, writes `pick: x` and leaves `lenses` without `a` and `b`, so `contains` `pick: x` and the file lacks `a: A`;
a standalone write of `pick=x` with `lenses.a=X` leaves `a: X` out of the file;
a standalone file holding `pick: x\n` takes `lenses.a=X` alone and leaves `a: X` out of the file, the selector read from the file's bytes;
a standalone `pick=nope` fails with `names no shipped layer` and removes the scaffold;
a hub worktree write of `pick=x` into an absent file fails with `--layer default`, `errLacks` `delete the pick line`, and leaves no file;
a hub worktree write of `lenses.a=Y` into a file holding `pick: x\n` fails with `delete the pick line from` and leaves the file as it was.
`TestEdit` gains an `overlay` row field swapping in `overlaySpec()` renamed `testmod`, and an `existing` field holding the worktree file's bytes before the call (empty keeps the loop's `original`).
Rows: a hub worktree edit that writes `pick: x\n` into an absent file fails with `--layer default` and leaves no file;
a hub worktree edit of an existing `lenses:\n  a: Z\n` file that writes `pick: x\n` fails and leaves the file as `lenses:\n  a: Z\n`;
a hub worktree edit of an existing `pick: x\n` file that writes `pick: x\nlenses:\n  a: Z\n` fails with `delete the pick line from` and leaves the file as `pick: x\n`;
a hub worktree edit of an existing `lenses:\n  a: Z\n` file that writes `lenses:\n  a: W\n` saves it.

`internal/configengine/doc.go`'s `# Template overlay` section adds the write side: the scaffold without per-name entries, the selector read from the document being written, and the `Set` and `Edit` refusals.
The header comments of `set.go` and `edit.go`, which no member glyph spans, say an overlay module's base scaffold holds empty per-name maps.
`internal/configcli/doc.go`'s `--set` paragraph says a hub worktree write picking a layer the base does not pick is refused naming `--layer default` from the prime.

**ImpactSummary:** `setSparse` has one caller, `Set`, re-signed on this card; every spec without an overlay writes, scaffolds and edits byte for byte as before, and `configcli`'s set and edit paths pass the new refusals through unchanged.
