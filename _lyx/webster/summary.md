# batten opens VS Code on the new pair with reed attached

All six batches reported `done`, with no deviations from the plan's file-ops; the digests carried only informational scope warnings.

- 01 (`9ca7bf75`): `vscode.WriteConfig` takes a `TaskChain` selector, adding attach-only next to interactive; `ideengine.Spawn` passes interactive, so plain `lyx ide spawn` is unchanged.
- 02 (`30fe441e`): `fabricengine.ExcludeAnchoredDir` writes an anchored `info/exclude` line under the existing lock, skipping when the path is already ignored.
- 03 (`21bb1e49`): `fabricengine.PathTracked` is a read-only tracked-path query.
- 04 (`c0dc72b4`): `ideengine.SpawnDriven` opens the driven pair, shares `resolveSpawnTarget` with `Spawn`, and skips the exclude write for tracked paths.
- 05 (`60d3caac`): `battenshed.InnerRunDeps.OpenIDE` is called once per run after a successful spawn, gated by an `-ide-opened` marker; a nil closure defaults to a no-op.
- 06 (`21b44d31`): battencli wires `OpenIDE` to `SpawnDriven`, the integration `TestMain` stubs `CodeLauncher`, and `docs/overview.md` is updated.

The integration suite (`## verify:`) ran once at HEAD `21b44d31` and reported `status: OK`.
