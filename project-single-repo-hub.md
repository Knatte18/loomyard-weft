# P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric

Rendered 2026-11-21 19:22 UTC; every age below is as of then.
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
| T3 | **Hardener writes its own charter, drives either hub layout**<br>• Hardener derives its campaign from the module, builds one-repo sandbox hubs, needs no hub gate writes<br>• `darn: writer opus[medium]`<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`hardener-self-charter`](design-hardener-self-charter.md) | darn · solid | starting · Seed-IDE | neve | 6m<br>0m | shed, burler, gate, stencil, fabric |

### Ready (0)

Waits on nothing open; can start now.

_None._

### Layer A (1)

Waits only on Running, Ready or Draft entries.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T2 | **Hardener campaign on fabricengine**<br>• Hardener run on internal/fabricengine, both hub layouts<br>• `hardener: packages ["internal/fabricengine"]` | [`harden-fabricengine`](design-harden-fabricengine.md) | hardener | T3 | 6m<br>0m | fabric |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
