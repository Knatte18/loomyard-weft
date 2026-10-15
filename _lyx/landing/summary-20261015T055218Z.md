# lyx config --set refuses a value the module's loader rejects

`lyx config loom --layer local --set 'plan=["opus[medium]", "sonnet[high]"]'` wrote `plan: '["opus[medium]", "sonnet[high]"]'` and reported `ok: true`.
`plan` takes one model-spec, so every later reader failed on the stored string (issue #620).
`--set` now checks the written layer with the module's own loader and refuses a value that loader rejects, naming the key, with the file left as it was.

## Behaviour

- Every `--set` write is checked: the worktree file, the local override, and the board-dir file of a hub-wide module or of `--layer default`.
- After the write, the module is loaded through its own loader.
  A worktree or local write loads the worktree's stack; a board-dir write loads the board file alone, after the strict `configengine.Load` check it already had.
- When the load fails, the file is restored to its previous bytes, or removed if the write created it.
  The error names the module, carries the loader's error, which names the key (for example `loom config key "plan": modelspec: ...`), and says the file is unchanged.
  A refused worktree write runs no fabric sync, and a refused board-dir write commits nothing.
- The load also runs before the write.
  A write is refused only when the load afterwards fails with an error text it did not fail with before.
  So a stack that already fails to load, such as one with no base layer or with a bad value in another layer, still takes a write that leaves that failure as it was, and a `--set` that repairs the bad key goes through.
- Per-round keys such as `plan_review` still take a quoted flow list and store it as a list.

## Design decisions

- `configreg.Module` gains `CheckLoad func(configengine.Dirs) error`, set on every row of `Modules()` through two small generic adapters, one for loaders that take only `dirs` and one for loaders that also take their module name.
  The loader is the one place that knows which values a module takes, so `--set` refuses exactly what a later reader would.
- The check writes first and rolls back, rather than validating an in-memory document, because the loaders read the layer stack from disk through `configengine` and none of them takes a document.
  It reuses `hubwide.go`'s `snapshotFile` and `rollBack`.
- A refusal needs a *new* failure so that one broken layer cannot block `--set` on the others.
  The bound: a new bad value masked by an earlier failure the loader reports first also gets through.
- No yamlengine shape rule was added for a bracketed value at a scalar key: the loader check covers that case and every other value a loader rejects, such as a malformed model-spec scalar.
- The interactive editor paths keep their current checks; the board entry asks for `--set`.
- Loom's loader also reads `burler.yaml` and `models.yaml`, so a loom write depends on those files loading too.
  The new-failure rule keeps an already-broken neighbour from blocking it.

## Docs

`internal/configcli/doc.go`, `lyx config --help` and the config entry of `docs/overview.md` describe the check and its bound.
To keep `lyx config --help` under its 2000-byte cap, the help's sentence on preserved keys was shortened.

## Tests

- `TestDispatchLayers` gains rows: a list given to loom's `plan` is refused on the local, worktree and default layers, naming the key, with no local or worktree file left, the board file unchanged and no sync.
  A refused local write restores a present file byte for byte, a valid `plan_review` list is still written as a flow list, and a write onto a local layer that already fails the same way still goes through.
- `TestNames` asserts every registered module sets `CheckLoad`.
