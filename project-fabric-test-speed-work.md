# P3 Take internal/fabricengine tier 1 time down: dotgit worktree list, hub templates, fewer git processes

Rendered 2026-11-07 16:52 UTC; every age below is as of then.
Under: [Board](README.md).

[`fabric-test-speed-work`](design-fabric-test-speed-work.md) · Age 2h · Owner `neve`

Groups the entries filed from the Sketch `fabric-test-speed`: hubforge-registered-templates, hubforge-slim-copy, git-config-from-go, dotgit-worktree-reader, hubreconcile-templates, loomcli-templates, battencli-templates, fabric-test-folds, fabric-anchor-origin-templates, fabric-livestate-merge-templates, copyhub-fixtures-and-hook, fabric-test-speed-remeasure.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T17 | **Re-measure tier 1 and internal/fabricengine after the fabric test speed tasks land** | [`fabric-test-speed-remeasure`](design-fabric-test-speed-remeasure.md)<br>[Sketch](decision-fabric-test-speed-remeasure.md) | darn · docs | Darn-Write | neve | 2h<br>2h | fabric, testkit, git |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T16 | **CopyHub in the NewHub fixtures of fabricengine and adoptcli, and the hook without dirname and basename** | [`copyhub-fixtures-and-hook`](design-copyhub-fixtures-and-hook.md)<br>[Sketch](decision-copyhub-fixtures-and-hook.md) | 2h<br>2h | fabric, testkit, git |
