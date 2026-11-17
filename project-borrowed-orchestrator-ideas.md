# P2 Ideas borrowed from comparable agent orchestrators

Rendered 2026-11-17 10:57 UTC; every age below is as of then.
Under: [Board](README.md).

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 7m · Owner `neve`

Ideas from the October 2026 survey of comparable agent orchestrators that lym has not built.
The note orchestrator-survey-ideas holds the remaining leads; each other member is one idea picked as most worth doing (orch, 2026-11-17).

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
| T14 | `draft` **Review rounds converge: fixer rebuttals, incremental re-review, stalemate on no tree change** | [`review-rounds-converge`](design-review-rounds-converge.md) | motif |  | 7m<br>0m | burler, shed, stencil |
| T16 | `draft` **A review lens that flags changes no decision asked for** | [`unrequested-scope-lens`](design-unrequested-scope-lens.md) | darn · light |  | 7m<br>0m | lenses, burler |
| T20 | `draft` **Refuse a commit that adds a secret file** | [`sensitive-path-commit-guard`](design-sensitive-path-commit-guard.md) | darn · light |  | 7m<br>0m | webster, fabric |

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F8 |  | **Flag an agent that edits the same file again and again** | [`edit-loop-detector`](design-edit-loop-detector.md) | 7m<br>7m | shuttle, friction |
| F12 |  | **Warn on resume when a run's stencils changed since it started** | [`stencil-pin-per-run`](design-stencil-pin-per-run.md) | 7m<br>7m | stencil, shed |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>7m | burler, shed, orch, quarry, report |
