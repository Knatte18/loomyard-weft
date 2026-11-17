# P2 Ideas borrowed from comparable agent orchestrators

Rendered 2026-11-17 10:51 UTC; every age below is as of then.
Under: [Board](README.md).

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 2m · Owner `neve`

Ideas from the October 2026 survey of comparable agent orchestrators that lym has not built.
The note orchestrator-survey-ideas holds the remaining leads; each other member is one idea picked as most worth doing (orch, 2026-11-17).

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (9)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Later review rounds check earlier findings, not the whole diff again** | [`incremental-rereview`](design-incremental-rereview.md) | 2m<br>2m | burler, stencil |

### Features (8)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F2 |  | **A per-run token and cost budget, audit mode first** | [`run-cost-budget`](design-run-cost-budget.md) | 2m<br>2m | shed, report |
| F5 |  | **A burler fixer may rebut a finding, and the next round answers it** | [`review-fixer-rebuttal`](design-review-fixer-rebuttal.md) | 2m<br>2m | burler, stencil |
| F6 |  | **A review lens that flags changes no decision asked for** | [`unrequested-scope-lens`](design-unrequested-scope-lens.md) | 2m<br>2m | lenses, burler |
| F7 |  | **Halt a segment after review rounds that change nothing in the tree** | [`stalemate-no-tree-change`](design-stalemate-no-tree-change.md) | 2m<br>2m | burler, shed |
| F8 |  | **Flag an agent that edits the same file again and again** | [`edit-loop-detector`](design-edit-loop-detector.md) | 2m<br>2m | shuttle, friction |
| F10 |  | **Refuse a commit that adds a secret file** | [`sensitive-path-commit-guard`](design-sensitive-path-commit-guard.md) | 2m<br>2m | webster, fabric |
| F12 |  | **Warn on resume when a run's stencils changed since it started** | [`stencil-pin-per-run`](design-stencil-pin-per-run.md) | 2m<br>2m | stencil, shed |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 4w<br>2m | burler, shed, orch, quarry, report |
