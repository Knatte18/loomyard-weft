# P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric

Rendered 2026-11-19 18:31 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 22h · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **Remove f06acf39's records-on-code-branch layout and revert its two-repository widenings**<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`single-repo-code-records-removal`](design-single-repo-code-records-removal.md)<br>[Sketch](decision-single-repo-code-records-removal.md) | darn · solid | Record-Write | neve | 1m<br>1m | fabric, cwd, cli, batten, loom, landing, git, board, docs, pattern |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | **One-repository hubs with records and board under origin's refs/lym/, the layout read only by fabric** | [`single-repo-namespace-layout`](design-single-repo-namespace-layout.md)<br>[Sketch](decision-single-repo-namespace-layout.md) | tapestry · solid | T8 | 1m<br>1m | fabric, git, docs, pattern |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
