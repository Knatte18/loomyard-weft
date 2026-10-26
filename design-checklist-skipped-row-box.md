# Checklist marks a row skipped by its entry gate as [-], not open

`checklist-skipped-row-box` · Task · enhancement, loom, shed · running · PR-Review

## Seen (operator, 2026-10-26)

With the Drift rows switched off in loom, a run's checklist in the overview leaves the Drift rows open, `[ ]`, after the run has passed them.
The rows did run: each row's entry gate came out false and shed moved on, so an open box reads as not yet reached when it was settled.
The boxes come from `internal/loomcli/checklist.go`, which knows only done (`[x]`) and not reached (`[ ]`).

## Idea

- A row the run passed because its entry gate was false gets its own box, `[-]`, marking it skipped rather than open.
- This holds for every row skipped by its entry gate, not only the Drift rows.
