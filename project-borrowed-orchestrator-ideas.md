# P2 Ideas borrowed from comparable agent orchestrators

Rendered 2026-11-17 12:35 UTC; every age below is as of then.
Under: [Board](README.md).

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 1h · Owner `neve`

Ideas from the October 2026 survey of comparable agent orchestrators that lym has not built.
The note orchestrator-survey-ideas holds the remaining leads; each other member is one idea picked as most worth doing (orch, 2026-11-17).

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Running (2)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T5 | **Warn on resume when a run's stencils changed since it started** | [`stencil-pin-per-run`](design-stencil-pin-per-run.md) | darn · light | Describe | neve | 1h<br>53m | stencil, shed |
| T6 | **Flag an agent that edits the same file again and again** | [`edit-loop-detector`](design-edit-loop-detector.md) | darn · light | landing · Finalize | neve | 1h<br>53m | shuttle, friction |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Sketches (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| S1 |  | **Review rounds converge: fixer rebuttals, incremental re-review, stalemate on no tree change** | [`review-rounds-converge`](design-review-rounds-converge.md)<br>[Sketch](decision-review-rounds-converge.md) | 1h<br>44m | burler, shed, stencil |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>1h | burler, shed, orch, quarry, report |
