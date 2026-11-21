# P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric

Rendered 2026-11-21 18:45 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 2d · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T4 | **One-repository sandbox: second clone and namespace hand check**<br>• Clone the sandbox again, land a task from it, and grep fabric and gitrepo for un-namespaced records refs<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`onerepo-sandbox-reclone`](design-onerepo-sandbox-reclone.md) | darn · docs | Code-Bouncer | neve | 1h<br>1h | fabric, git |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
