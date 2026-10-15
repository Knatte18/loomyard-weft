# Card 8 — config-key-move

**Create:**
- `plan:internal/configreg#KeyMove` -> `type KeyMove struct`
- `plan:internal/configreg#KeyMoves` -> `func KeyMoves() []KeyMove`
- `plan:internal/configreg#moveRunSettings` -> `func moveRunSettings(settings map[string]any) map[string]any`
- `plan:internal/configsync#moveKeys` -> `func moveKeys(dir, seedDir string, layer configengine.Layer, base, apply bool) ([]Result, error)`
- `plan:internal/configsync#templateLineComment` -> `func templateLineComment(template []byte, key string) string`
- `plan:internal/configsync#TestReconcile_MovesKeysAcrossModules` -> `func TestReconcile_MovesKeysAcrossModules(t *testing.T)`

**Edit:**
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configsync#Result`
- `internal/configsync#ReconcileAll`
- `internal/configsync#ReconcileHubWideAt`
- `internal/hubreconcile#logResult`
- `internal/hubreconcile_test#TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`
- `internal/configcli#runReconcile`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `internal/configsync#TestReconcile_FailureNamesItsFile`
- `internal/fabriccli#CloneAndWire`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/configreg#Lookup`
- `internal/hubreconcile#snapshotConfig`
- `internal/configsync#findLayerResult`
- `internal/configsync#writeModuleFile`

**Intent:**
Carry the old `shuttle.yaml` values into `claude.yaml` through a declared cross-module key move (D4), applied by every reconcile entry point and by run-settings preparation, so reconcile never silently drops an operator's binary path, switched-off Agent deny or role TTLs.

`configreg.KeyMove` names a source module and key and a target module and key; `KeyMoves` returns the five rows of D1's table, `shuttle` to `claude`: `claude` to `binary`, `claude_deny_agent_tool` to `deny_agent_tool`, `claude_deny_ask_user_question` to `deny_ask_user_question`, `claude_prompt_cache_ttl` to `prompt_cache_ttl`, `claude_prompt_cache_ttl_roles` to `prompt_cache_ttl_roles`.
The table stays after this task, since any layer or run-settings entry not yet reconciled still meets it.

`configsync.moveKeys` applies the table to one layer: `dir` is the board dir or anchor, `layer` the layer whose files it reads (`LayerLocal` forms paths with `configengine.LocalFile`, every other layer with `configengine.ConfigFile`), and `base` marks the full base layer.
For each row whose source key the layer's source file holds, it writes the value, unexpanded, into the target module's file of the same layer under the target key, with its line comment replaced by `templateLineComment`'s answer for the target template, and removes the key from the source file:
- an absent target file is created, at a base layer as the full target template carrying the moved values, at a sparse layer holding only the moved keys;
- at a base layer, a target already holding the key with a value other than the target template's default keeps its value and the source key is dropped, a kept-target collision; a target value equal to the template default is overwritten by the source value, an overwrite;
- at a sparse layer, any value the target already holds is kept and the source key dropped, a kept-target collision;
- a source file the move empties is deleted, as a pruned sparse file is today.

`seedDir` is set only at the board pass of `ReconcileHubWideAt` under `SeedDefaults`, to the prime's anchor, and empty everywhere else.
When the board's source file is absent and `seedDir` is set, the source is the prime's copy the module loop would seed it from (absent, empty or unparseable meaning no source, as `primeConfig` reads it); the move then never writes that board source file, since the loop seeds it from the prime and prunes the moved keys, and an absent board target is created from the prime's copy of the target when that copy parses, else from the target template, carrying the moved values.
So a board default seeded from the prime in the same pass carries the operator's moved values.

A source or target file the move reads that cannot be read or parsed fails the move with a `*FileError` naming that file's module and path, before anything is written, so the failure names the file just as the module loop's would.

It returns one `Result` per file it changes or would change, with `Module` and `Layer` set and `Applied` when it wrote; `Result` gains `Moved`, `KeptTarget` and `Overwritten`, each a sorted list of lines, set on a dry run too.
A `Moved` line reads `<source module>.<source key> -> <target module>.<target key>`; a `KeptTarget` line adds both unexpanded values, `<source>: <source value> dropped, <target>: <target value> kept`, and an `Overwritten` line `<source>: <source value> replaced <target>'s template default <default value>`, so the operator can recover a dropped value from the report.
`Result`'s doc, which says it reports a single module's reconcile, says a move result reports one file the move changed and that a module may appear in a move result and its own reconcile result.
With `apply` false it writes nothing and reports the pending moves.
A second pass over a moved layer finds no source key and returns no result, so the move is idempotent.
Bound: the move only relocates values between two registered modules' files of one layer; it never changes a value and never crosses layers, and at the local layer it can carry only what the local rule admitted in the source, which after card 4 is the TTL keys, since the old Executable and Guard keys were already refused there.
Every path is formed through `configengine`'s path functions and every write goes through `fsx.AtomicWriteBytes`, as reconcile writes today (`PATTERN-config-layers`); the move runs only inside reconcile, never from a load (`PATTERN-binary-change-reconcile`).

Each reconcile entry point runs the move as a layer's first step, before any module of that layer is seeded, materialized or reconciled, since `claude` sorts before `shuttle` in the registry:
- `ReconcileHubWideAt` runs it at the board dir as a base `LayerDefault` before its module loop, with `seedDir` the prime's anchor under `SeedDefaults`, so a repo default `claude.yaml` the move creates is the one the loop then reconciles, and a board default seeded in the same pass carries the moved values;
- `ReconcileAll` runs it at the anchor's worktree layer, as a base layer standalone (`boardDir` empty) and a sparse one in a hub, and at the local layer, before its module loop.
The move's results lead the returned slice.
`hubreconcile` needs no new path: its snapshot already covers every registry module's files, `claude` included, so a walk that fails after the move restores both files, and it commits every applied worktree-layer or board result, the move's included; `logResult` logs `moved`, `keptTarget` and `overwritten` beside the other fields.
`configcli`'s `runReconcile` renders each result's `Moved`, `KeptTarget` and `Overwritten` lines like its `Migrated` lines, so `lyx config reconcile` without `--apply` reports each pending move.
`fabriccli`'s `CloneAndWire` comment that says `ReconcileAll` returns one result per registered module is corrected to say the move's results lead the slice; its loop already records every applied worktree-layer result's file, the move's included.

`PrepareRunSettings` runs `moveRunSettings` before its module loop: each source module's object loses every moved key, which lands in the target module's object under the target key unless that object already holds it, in which case the target's value is kept and the source key dropped, so `shuttle: {claude_prompt_cache_ttl_roles: …}` becomes `claude: {prompt_cache_ttl_roles: …}` and the emptied `shuttle` object is dropped as an empty one is today; the target module's own migrate and validation then judge the moved keys.
A source module whose value is not an object passes through untouched, so the module loop still refuses it as today.

Tests:
- `TestReconcile_MovesKeysAcrossModules` runs table rows over temp dirs with `configsync`'s own fixtures: a repo default, a worktree and a local layer each moving a key into a created `claude.yaml` (full at the base, sparse above it) with the source key gone; a pre-existing base `claude.yaml` holding template values plus a `shuttle.yaml` with `claude: /x` resolving `binary` to `/x` through `configreg.Lookup("claude")`'s spec and reporting an overwrite; a base target holding a non-default value keeping it and reporting a kept-target collision; a worktree `claude.yaml` holding a template-default value keeping it and reporting the collision; a board default `claude.yaml` absent before `ReconcileHubWideAt` with `SeedDefaults` and carrying the moved value after it, once with the source key in the board's own `shuttle.yaml` and once with the board's `shuttle.yaml` absent and the source key only in the prime's `shuttle.yaml`, after which the seeded board `shuttle.yaml` holds no `claude` key and the board `claude.yaml` holds the moved `binary`; a standalone `ReconcileAll` (`boardDir` empty) creating a full worktree `claude.yaml` from the template with the moved value, and overwriting a template-default target value; the `KeptTarget` and `Overwritten` lines carrying both values; a dry run writing nothing and reporting the pending moves; a second pass changing nothing; a `shuttle.local.yaml` the move empties being removed; and a local `claude_prompt_cache_ttl: {a: 1}` moved verbatim, after which the `claude` module's `CheckLoad` refuses naming `claude.local.yaml`.
- `TestPrepareRunSettings`'s shuttle open-map row now expects a `claude` document `prompt_cache_ttl_roles:\n    worker: 1h\n` and no `shuttle` document; a row with `shuttle: {claude_prompt_cache_ttl: 1h}` beside `claude: {prompt_cache_ttl: 5m}` expects a `claude` document holding `prompt_cache_ttl: 5m`; a row with `shuttle: "fast"` is still refused as a non-object value for module `shuttle`.
- `TestReconcile_FailureNamesItsFile` gains the row `ReconcileAll on an unparseable shuttle.yaml`, module `shuttle` in the base, which the move reads first and fails naming that file.
- `TestConfigReconcileInGitRepo` gains an assertion that a dry run over a worktree whose `shuttle.yaml` holds `claude: /x` reports the pending move line and leaves both files unchanged.
- `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` gains a row whose pair commits a `shuttle.yaml` holding `claude: /x` and an unparseable `loom.yaml`: the walk moves the key into `claude.yaml`, then fails on `loom.yaml`, and the row's existing assertions prove both files are restored and the tree is as it was; its repair commits the loom template, and the repaired walk commits the moved key.
- `cmd/lyx/testdata/test-budget.yaml` raises `internal/configsync` to 9.

**ImpactSummary:** Adds a pre-pass to every reconcile entry point and to run-settings preparation, plus three `Result` fields read by `hubreconcile`'s log and `configcli`'s report; a layer holding no source key behaves exactly as before.
