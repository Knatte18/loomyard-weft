# P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric

Rendered 2026-11-21 18:14 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 2d · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T3 | **One-repository sandbox: run tasks to landing**<br>• Run sequential, concurrent and abandoned tasks in the sandbox hub and check origin after each<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`onerepo-sandbox-tasks`](design-onerepo-sandbox-tasks.md) | darn · docs | Darn-Write | neve | 45m<br>45m | fabric, landing |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T4 | **One-repository sandbox: second clone and namespace hand check**<br>• Clone the sandbox again, land a task from it, and grep fabric and gitrepo for un-namespaced records refs<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`onerepo-sandbox-reclone`](design-onerepo-sandbox-reclone.md) | darn · docs | T3 | 45m<br>45m | fabric, git |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
