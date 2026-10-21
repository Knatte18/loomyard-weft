---
verdict: APPROVED
---

## Measure

The diff of pull request 782 at `8740e7a2` against `main` is measured against board entry `lenses-template-frozen`.
The decision record holds no `Added after Discussion` entry, so the board body is the measure unamended.

## Board points against the diff

- Drop a lens or fan equal to the template's current value, reporting each drop per name: `mergePerNameMaps` in `internal/yamlengine/reconcile.go` leaves out a template-equal name and returns it in a new sorted `dropped`; `configsync.Result.Dropped` carries it at both base-layer reconcile sites, `lyx config reconcile` emits `dropped`, and the `hubreconcile` walk log carries it.
- Equal means the same lens prose or the same ordered fan list: `sameValue` compares decoded values with `reflect.DeepEqual`, so list order counts and scalar style does not; a `TestReconcile` row pins a quoted and folded scalar and a block list as equal.
- A dropped name resolves from the template at load: the drop removes only a value equal to the template's, and the load's fill supplies the name.
- No more copying of template names the file lacks: the merge skips a missing template name and reports nothing for it; an absent or null map is written as an empty mapping.
- Never overwrite a held value, and migrations still write what they move: tuned and file-only names are kept as written, and the key-move and migration paths are untouched.
- Generic over every per-name map in `yamlengine` and `configsync`: the change keys off `PerNameMaps`, not `lenses`.
- The `internal/lenses/template.yaml` header and `internal/lenses/doc.go` state the new behaviour; `internal/yamlengine/doc.go`, `internal/configengine/doc.go`, `internal/configcli/doc.go`, `internal/configreg` and the `docs/overview.md` module row follow.
- Not in scope, respected: no change treats an entry equal to an older template value as droppable, and the hub's stale `twill-plan` is left for the orch.

Two changes go past the board body's literal wording, and both serve its goal.
The base-layer `SetValues` drop keeps a `--set` from writing every template name back into the file, which would undo the goal that the file holds only what the hub changed.
The load log's omission of filled per-name keys follows from that goal making a sparse base file the normal state.
Neither is a scope deviation.

## Open risks carried over

- Darn-Review round 1's `filled-keys-log-filter-untested` is addressed by commit `d13b4618`, which adds the test in `internal/configengine/config_test.go`.
- No carried-over risk names an unrecorded parent correction.

## Overlap

- `lyx loom peers` lists no parent commits.
- Peer `origin-reconcile` (row 11) and peer `run-reentry` (row 9) list no changed paths, and neither decision record names `yamlengine`, `configsync`, `configengine`, `hubreconcile`, lenses or `PerNameMaps`.
  No overlap with either peer, so no landing order applies.
- Board entry `lenses-language-sets` is an idea, not a run in flight; it seeds per-hub lens sets and neither duplicates nor contradicts this run.

## Superseded

None: no other board entry is made obsolete by this run.
