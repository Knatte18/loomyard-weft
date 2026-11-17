# Later review rounds check earlier findings, not the whole diff again

`incremental-rereview` · Note · enhancement, burler, stencil

## Seen (orch, 2026-11-17)

Each review round after the first reviews the whole diff again, so it can raise new findings on code it passed before.
Bounce rounds are a large part of a run's cost.
Source: the survey F17 (`orchestrator-survey-ideas`), from Traycer's re-verify, which checks only earlier findings and marks each resolved or outdated,
and Fusion, which forbids new non-blocking findings on a re-review.

## Idea

- A later round checks the earlier round's findings against the fixes and marks each resolved, still open or outdated.
- It raises a new finding only on code the fixes changed, and only at BLOCKING severity.
