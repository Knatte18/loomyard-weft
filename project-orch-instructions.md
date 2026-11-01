# P1 Orch instructions in two layers: lym's role and the repository's rules in heddle

Rendered 2026-11-01 15:58 UTC; every age below is as of then.
Under: [Board](README.md).

`orch-instructions` · Age 2h · Owner `neve`

One fixed orch role for every repository, repository orch rules in heddle through lym orch rules, and the orch note for run state only

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Running (1)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T11 | **One fixed orch role for every repository, brought up to date**<br>• The rules for every orch move into the shipped role, which settles run-level calls itself and has no board copy | [`orch-role-current`](design-orch-role-current.md) | darn | Code-Burler-Fix | neve | 2h<br>1h | orch, stencil |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
