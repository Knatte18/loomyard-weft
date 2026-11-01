# P1 Orch instructions in two layers: lym's role and the repository's rules in heddle

Rendered 2026-11-01 14:57 UTC; every age below is as of then.
Under: [Board](README.md).

`orch-instructions` · Age 1h · Owner `neve`

One fixed orch role for every repository, repository orch rules in heddle through lym orch rules, and the orch note for run state only

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (3)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | `draft` **lym's design invariants as PATTERN entries**<br>• doc.go as documentation only, fabric's isolation, no hard-coded names and the other code invariants become PATTERN entries | [`pattern-design-principles`](design-pattern-design-principles.md) | darn |  | 1h<br>0m | pattern |
| T10 | `draft` **Heddle gains an orch-prompt target, and lym orch rules stores in heddle**<br>• A heddle target for orch sessions only replaces the hub-wide and local orch rules files<br>• `profile: solid` | [`heddle-orch-target`](design-heddle-orch-target.md) | darn |  | 1h<br>0m | heddle, orch |
| T11 | `draft` **One fixed orch role for every repository, brought up to date**<br>• The rules for every orch move into the shipped role, which settles run-level calls itself and has no board copy | [`orch-role-current`](design-orch-role-current.md) | tapestry |  | 1h<br>1h | orch, stencil |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
