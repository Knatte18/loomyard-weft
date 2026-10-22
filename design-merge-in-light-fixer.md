# Merge-In with its own light fixer and no Merge-Review perch

`merge-in-light-fixer` · Note · enhancement, loom, burler, fabric

## Seen

Merge-In already has its own fixer: `internal/mergeresolve` spawns a session that resolves textual conflicts and may edit other files to keep the merged tree building.
After it come Merge-In's verify, then the Merge-Review perch (Bouncer, review, fix) over how the task's code meets what the parent gained.
A verify failure after the merge halts the run, and batten sends it back to Webster-Burler-Review.
Audit `pr-rework-audit`: Merge-Review ran in 73% of runs, its fixer changed code in a third of those, about $0.96 per run; one of its fixes was a scope deviation PR-Align later rejected.

## Idea (operator, 2026-10-22)

Merge-In needs no perch, only its own light fixer.

- Drop the Merge-Review perch.
- A verify failure after the merge goes to Merge-In's own light fixer (the merge-resolve session, or one like it on a light model), which fixes it in place and re-runs the verify, instead of a trip back to Webster-Burler-Review.
- PR-Align's check against landed parent commits stays the semantic check.
