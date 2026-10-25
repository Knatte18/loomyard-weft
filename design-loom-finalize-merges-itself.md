# Loom Finalize resolves a moved parent itself instead of rerouting to Parent-Align

`loom-finalize-merges-itself` · Note · enhancement, loom, batten

## Seen (umbrella-anchor, 2026-10-25)

A loom Finalize found that the parent's changes since Parent-Align touched files the task changed (`docs/overview.md`, `internal/beamcli/cli.go`, `internal/beamcli/doc.go`), judged the merge large and rerouted to Parent-Align.
The reroute re-enters the whole Code-Review perch over the whole diff, which takes long enough for the parent to move again, so a run can loop through Parent-Align and the perch several times, each loop a full perch's cost.
A loom Finalize aborts every conflict and reroutes (`ConflictReroutes` in `internal/loomcli/landingdeps.go`), while a darn Finalize already resolves a conflict with the conflict session under the parent lock, and a beam's Finalize already lets a large merge go on to the landing gate (`LargeMergeLands`).

## Idea (operator, 2026-10-25)

- A loom Finalize handles a moved parent itself: it resolves a conflict with the conflict session under the parent lock, as darn's does, and a large merge goes on to the landing gate and its merge fixer, as a beam's does.
- A conflict or overlap in Finalize no longer reroutes to Parent-Align or re-runs the Code-Review perch.
- Parent-Align, before Code-Review, stays the row that catches the bulk of the parent's drift; Finalize's agent takes only what moved since.
