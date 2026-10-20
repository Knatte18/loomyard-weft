# Per-name maps keep only tuned entries, so template edits reach hubs

## What changes

A hub's `lenses.yaml` used to hold every template lens and fan word for word, because reconcile copied each template name the file lacked into it and kept every name the file held.
No later template edit to a lens or fan reached a hub until the operator deleted that entry by hand.

Reconcile now writes only what the file changed into each per-name map declared through `configengine.Spec.PerNameMaps`, so the change covers every per-name map, not only the `lenses` module's two:

- A template name the file lacks is no longer copied in or reported added; the load's template fill resolves it, as it already did.
- A template name the file holds with the template's current value is left out and reported per name as `P.<name>` in a new `dropped` return of `yamlengine.Reconcile`, so the loaded value is unchanged and the next template edit to that name reaches the file.
  Equal means the two values decode to the same data, so a block scalar and a quoted string with the same text, or a flow and a block list with the same entries, count as equal.
- A name whose value differs from the template's, and a name the template lacks, are kept as before.
- An absent or null per-name map is written as an empty mapping and `P` is reported added, since a full base layer holds every template key and the load's fill refuses a null where the template holds a mapping.

`configsync.Result` gains `Dropped`, set at both base-layer reconcile sites, the hub file reconcile and the standalone worktree base, on a dry run too.
A non-empty `Dropped` makes `--apply` rewrite the file, like `Added` does.
`lyx config reconcile` reports it as `dropped` when non-empty, and the hub reconcile walk logs it.
Key moves and `Migrate` hooks still write what they move; the drop then applies to the resulting file like any other.

## Design decisions

- **The base-layer `--set` drops template-equal names too.**
  `yamlengine.SetValues` builds its output from the template, so without this a `--set` of any key in `lenses.yaml` would write every template lens and fan back into the file and freeze them again.
  After its pairs apply, it leaves out every per-name map name holding the template's value, including one a pair just set to it; the loaded value is unchanged.
  The board entry names only reconcile, and this reaches one step further to meet its goal.
- **Sparse layers never drop.**
  `yamlengine.Prune` is unchanged: on a worktree or local layer a template-equal name can override a tuned value in the layer below, so dropping it would change what loads.
- **The load stops logging per-name names it fills.**
  `configengine`'s "filled missing keys from template" Info line now leaves out keys under a declared per-name map, since a base file lacking template names is now the normal state and would log on every load.
  A filled per-name map path itself is still logged.

## Docs

The `lenses.yaml` header in `internal/lenses/template.yaml` and `internal/lenses/doc.go` now state that the file holds only tuned or added entries, that reconcile never copies a template entry in and drops a template-equal one, and that a deleted template entry resolves from the template at load without being written back.
`internal/yamlengine/doc.go`, `internal/configengine/doc.go` and the `internal/lenses` row of `docs/overview.md` follow.

## Not in scope

An entry equal to an older template value is a tuned entry to reconcile and stays.
This hub's `twill-plan`, which equals the old five-lens template fan, is deleted by hand after landing.

## Tests

- `TestReconcile` rows pin the drop, style-insensitive equality, the kept tuned and file-only names in order, the empty mapping for an absent or null map, and that a second reconcile adds, drops and removes nothing.
- `TestSetValues` rows pin that a `--set` leaves template names out and drops a name set to its template value, and `configengine`'s `TestSet` pins the written base file exactly.
- The `configsync` lenses rows pin `Dropped` on a dry run and an apply, the rewritten file, a `lenses.Load` that still resolves every template lens and fan, and the key-move rows' cut files.
- The `configcli` integration tests pin `dropped` in the `--layer default` JSON and its absence on a sparse worktree layer holding a template-equal fan, and the `hubreconcile` integration test pins that a moved `burler.yaml` no longer has the template's untuned lenses copied in.
