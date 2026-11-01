# P1 Orch instructions in two layers: lym's role and the repository's rules in heddle

Rendered 2026-11-01 15:38 UTC; every age below is as of then.
Under: [Board](README.md).

`orch-instructions` · Age 2h · Owner `neve`

One fixed orch role for every repository, repository orch rules in heddle through lym orch rules, and the orch note for run state only

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (1)

Concrete and claimable; only a task can run.

### Ready (1)

Waits on nothing open; can start now.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **One fixed orch role for every repository, brought up to date**<br>• The rules for every orch move into the shipped role, which settles run-level calls itself and has no board copy | [`orch-role-current`](design-orch-role-current.md) | darn |  | 2h<br>40m | orch, stencil |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.

## Done (1)

Finished, awaiting `lym board prune --apply`.

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T10 | **Heddle gains an orch-prompt target, and lym orch rules stores in heddle**<br>• A heddle target for orch sessions only replaces the hub-wide and local orch rules files<br>• `profile: solid` | [`heddle-orch-target`](design-heddle-orch-target.md) | 2h<br>40m | heddle, orch |
