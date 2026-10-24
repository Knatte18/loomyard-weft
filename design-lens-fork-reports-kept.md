# Every lens fork report is kept verbatim in the run records

`lens-fork-reports-kept` · Task · enhancement, burler, loom · running · Worktree-Create

## Seen (orch, 2026-10-22)

A twill writer's lens forks each return a report, and the lead writes one line per finding into `_lyx/plan/lens-review.md`: the lens names, the severity, a short summary and how it was fixed.
The fork reports themselves are not kept in the run's records.
For a lens audit (which lenses overlap, which findings are real) the one-liners are too terse; the full reports survive only in the machine-local Claude Code transcripts under `~/.claude/projects/`, which are not in git and do not exist for runs from another machine.

## Idea (operator, 2026-10-22)

Every lens fork's report is saved verbatim in a support file of its own in the run's records, beside `lens-review.md`, for the Plan and Discussion writers' twill fans and the reviewer and fixer fans alike, so later audits read what each lens found from the archive tags.
Its format is pinned in `contracts/specs/`.
