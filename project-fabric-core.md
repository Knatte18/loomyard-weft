# P4 Fabric split into a git-free core, a layout package and a thin git engine

Rendered 2026-11-27 06:31 UTC; every age below is as of then.
Under: [Board](README.md).

[`fabric-core`](design-fabric-core.md) · Age 8h · Owner `neve`

Groups the entries filed from the Sketch `fabric-git-free-core`: fabric-core-foundation, fabric-core-push-origin, fabric-core-teardown, fabric-core-merge-landing, fabric-core-pull-replay, fabric-core-add-clone, fabric-core-records-guards.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T6 | **Fabric core foundation: layout and core packages, snapshot, action list, executor loop and the layout retarget** | [`fabric-core-foundation`](design-fabric-core-foundation.md)<br>[Sketch](decision-fabric-core-foundation.md) | tapestry · solid | Plan-Write | neve | 8h<br>8h | fabric, testkit, pattern, cwd, loom, batten, landing, gate, orch, board, webster, drift, report |

## Draft (6)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T23 | `draft` **Fabric core: push classification, retries, origin reconcile and failed-push records move to the core** | [`fabric-core-push-origin`](design-fabric-core-push-origin.md)<br>[Sketch](decision-fabric-core-push-origin.md) | tapestry · solid | T6 | 8h<br>8h | fabric, testkit, shed, board, pattern |
| T24 | `draft` **Fabric core: remove, destroy, cleanup, prune, remote teardown and pending teardown move to the core** | [`fabric-core-teardown`](design-fabric-core-teardown.md)<br>[Sketch](decision-fabric-core-teardown.md) | tapestry · solid | T23 | 8h<br>8h | fabric, testkit, shed, landing |
| T25 | `draft` **Fabric core: merge-in, mid-merge, handover, takeover, integrate, task patch and discard move to the core** | [`fabric-core-merge-landing`](design-fabric-core-merge-landing.md)<br>[Sketch](decision-fabric-core-merge-landing.md) | tapestry · solid | T24 | 8h<br>8h | fabric, testkit, shed, batten, landing |
| T26 | `draft` **Fabric core: pull, replay, bolt, reconcile, parent tips and warp forward move to the core; stencil reads leave fabric** | [`fabric-core-pull-replay`](design-fabric-core-pull-replay.md)<br>[Sketch](decision-fabric-core-pull-replay.md) | tapestry · solid | T25 | 8h<br>8h | fabric, testkit, shed, stencil, board, heddle |
| T27 | `draft` **Fabric core: add, clone, adopt, junctions, weft wiring and the hub workspace move to the core; the vscode import is cut** | [`fabric-core-add-clone`](design-fabric-core-add-clone.md)<br>[Sketch](decision-fabric-core-add-clone.md) | tapestry · solid | T26 | 8h<br>8h | fabric, testkit, ide, batten, heddle, reconcile, shed |
| T28 | `draft` **Fabric core: anchors, records store, commit, trailers and index move to the core; import guards, PATTERN close-out and remeasure** | [`fabric-core-records-guards`](design-fabric-core-records-guards.md)<br>[Sketch](decision-fabric-core-records-guards.md) | tapestry · solid | T27 | 8h<br>8h | fabric, testkit, pattern, docs |
