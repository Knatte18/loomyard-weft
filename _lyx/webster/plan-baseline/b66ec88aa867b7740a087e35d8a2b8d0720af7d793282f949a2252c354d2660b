# Card 10 — config-cli-sparse-hub-base

**Edit:**
- `internal/configcli#editHubWide`
- `internal/configcli#reconcileDefaultLayer`
- `internal/configcli#reconcileEnvelope`
- `internal/configcli/doc.go`
- `docs/overview.md`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configcli#TestConfigOverRealHub`

**Uses:**
- `internal/configcli#copyStagedToHub`
- `internal/configengine#BaseScaffold`
- `internal/configengine#SparseBaseHeader`
- `internal/configengine#BaseSparseMarker`

**Intent:**
`lyx config`'s board-dir writes keep a hub base sparse (D4) and its docs state it (D6).

`editHubWide`, the editor path of every `--layer default` edit and every hub-wide module's edit, stages an absent hub base as `configengine.BaseScaffold` over the board dir and the module's spec returns, `configengine.SparseBaseHeader` instead of the template when the module is not seed-only, and passes that same scaffold as `copyStagedToHub`'s `template` argument, so the changed-while-open check compares against the bytes the staging copy started from; a seed-only module's absent file, `profiles`', still stages the template.
A save that keeps the header writes a key-less hub base and commits it, since the empty-save abort fires only on an empty or whitespace-only result.
`--set` on the default layer already writes sparse through `configengine.Set` (card 4), so `setHubWide` needs no change.
`reconcileDefaultLayer`'s comment says the reconcile seeds absent bases sparse, never fills a present one, and reports the base strip of an unmigrated board as pending without performing it, and `reconcileEnvelope`'s comment on `stripped` says a default-layer result carries it for an unmigrated board.
When any default-layer result carries a pending `stripped` list, `reconcileEnvelope` adds a top-level `base_strip` field saying the strip is performed only by the hub reconcile of a start verb, such as `lyx loom resume` or `lyx batten run`, and never by this command, so an operator who sees the pending strip after `--apply` has the way forward.

`internal/configcli/doc.go` drops "Under a board the base layer is the repo default and stays full": every file layer of a non-seed-only module is sparse in a hub, the repo default included, so `--set` on any layer writes only the keys set and an editor edit of an absent base scaffolds the header; the `--layer default` sentence's clause "through the call the hub reconcile after a binary change makes" is replaced, since the walk's call strips and this one does not: `lyx config reconcile --layer default` seeds and prunes but never fills a base, and its dry run and apply list the one-time base strip under `stripped` as pending, which only the hub reconcile a start verb runs performs; `--apply` writes no board marker.
`docs/overview.md`'s config bullet replaces its `--layer default` "does the same" sentence: `lyx config reconcile --layer default` seeds absent board files sparse, prunes but never fills a present one, and lists the one-time base strip as pending, which only a start verb's hub reconcile performs; with `--apply`, run from the prime, it commits them in `_board`.

`TestDispatchHubWideBoard` gains rows: an editor edit of the absent loom default shows the editor `SparseBaseHeader`, and a save adding `review_max_bounces: 2` under it commits a board file holding only the header and that unit; an editor that saves the header unchanged commits a key-less base that `lyx config loom --print --layer default` prints as the header; an editor edit of an absent `profiles` file, seed-only, still shows the template; and an editor whose run creates the absent loom default on the board before it saves is refused with the changed-while-open answer, the board file keeping the other writer's bytes.
`TestConfigOverRealHub` gains a subtest over its hub with the board's loom default rewritten to carry the current template whole and the marker removed: the dry run of `reconcile --layer default` lists loom's template-equal keys under `stripped` and carries `base_strip`, and `--apply --layer default` leaves the loom default's units in place and writes no `configengine.BaseSparseMarker`.

**ImpactSummary:** the hub-wide editor's scaffold and the config CLI's comments and package doc; `lyx config <module> --layer default` and every hub-wide edit of an absent file change, a present file's edit is unchanged.
