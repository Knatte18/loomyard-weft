# P2 Ideas borrowed from comparable agent orchestrators

Rendered 2026-11-17 11:43 UTC; every age below is as of then.
Under: [Board](README.md).

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 54m · Owner `neve`

Ideas from the October 2026 survey of comparable agent orchestrators that lym has not built.
The note orchestrator-survey-ideas holds the remaining leads; each other member is one idea picked as most worth doing (orch, 2026-11-17).

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (4)

Concrete and claimable; only a task can run.

### Running (4)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Warn on resume when a run's stencils changed since it started** | [`stencil-pin-per-run`](design-stencil-pin-per-run.md) | darn · light | Darn-Write | neve | 54m<br>1m | stencil, shed |
| T6 | **Flag an agent that edits the same file again and again** | [`edit-loop-detector`](design-edit-loop-detector.md) | darn · light | Darn-Write | neve | 54m<br>1m | shuttle, friction |
| T14 | **Review rounds converge: fixer rebuttals, incremental re-review, stalemate on no tree change** | [`review-rounds-converge`](design-review-rounds-converge.md) | motif | Discussion-Burler-Fix | neve | 54m<br>46m | burler, shed, stencil |
| T20 | **Refuse a commit that adds a secret file** | [`sensitive-path-commit-guard`](design-sensitive-path-commit-guard.md) | darn · light | Code-Bouncer | neve | 54m<br>46m | webster, fabric |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (1)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>54m | burler, shed, orch, quarry, report |
