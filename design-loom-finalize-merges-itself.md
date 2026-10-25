# Loom and darn Finalize resolve a moved parent themselves instead of rerouting

`loom-finalize-merges-itself` · Note · enhancement, loom, batten

## Seen (umbrella-anchor, 2026-10-25)

A loom Finalize found that the parent's changes since Parent-Align touched files the task changed (`docs/overview.md`, `internal/beamcli/cli.go`, `internal/beamcli/doc.go`), judged the merge large and rerouted to Parent-Align.
The reroute re-enters the whole Code-Review perch over the whole diff, which takes long enough for the parent to move again, so a run can loop through Parent-Align and the perch several times, each loop a full perch's cost.
Loom, darn and beam run the same Finalize (`landingshed.Finalize`), with its conflict session and its merge fixer; only two flags set per recipe in `internal/loomcli/landingdeps.go` differ:

| Recipe | Conflict | Parent changes overlap the task's files |
|---|---|---|
| loom | aborts and reroutes to Parent-Align (`ConflictReroutes`) | reroutes to Parent-Align and the whole perch |
| darn | the conflict session resolves it in Finalize | reroutes to Code-Bouncer, whose perch reviews the overlap note |
| beam | the conflict session resolves it in Finalize | lands through the gate (`LargeMergeLands`) |

## Idea (operator, 2026-10-25)

- Loom and darn handle a moved parent in Finalize itself: the conflict session resolves a conflict under the parent lock, and an overlapping merge goes on to the landing gate, whose merge fixer fixes what fails.
- A conflict or overlap in Finalize no longer reroutes, so neither recipe re-runs its Code-Review perch for it.
- Parent-Align, before Code-Review, stays the loom row that catches the bulk of the parent's drift; Finalize takes only what moved since.
- A beam's landing is the opposite case and gets the full treatment instead; that belongs to the beam lifecycle task's `pawl` recipe.
