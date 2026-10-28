# A landing merges a moved parent in Finalize and sends a fixer escalate to the arbiter first

`landing-settles-in-place` · Task · enhancement, loom, batten, shed

## Goal

A landing settles what it can in place instead of rerouting or handing the stop straight to the orch:
a moved parent is merged in Finalize itself, and a merge fixer's `escalate` goes to the arbiter first.

## Finalize handles a moved parent itself (was note `loom-finalize-merges-itself`)

### Seen (umbrella-anchor, 2026-10-25)

A loom Finalize found that the parent's changes since Parent-Align touched files the task changed (`docs/overview.md`, `internal/beamcli/cli.go`, `internal/beamcli/doc.go`), judged the merge large and rerouted to Parent-Align.
The reroute re-enters the whole Code-Review perch over the whole diff, which takes long enough for the parent to move again, so a run can loop through Parent-Align and the perch several times, each loop a full perch's cost.
Loom, darn and pawl run the same Finalize (`landingshed.Finalize`), with its conflict session and its merge fixer; only two flags set per recipe in `internal/loomcli/landingdeps.go` differ:

| Recipe | Conflict | Parent changes overlap the task's files |
|---|---|---|
| loom | aborts and reroutes to Parent-Align (`ConflictReroutes`) | reroutes to Parent-Align and the whole perch |
| darn | the conflict session resolves it in Finalize | reroutes to Code-Bouncer, whose perch reviews the overlap note |
| pawl | the conflict session resolves it in Finalize | lands through the gate (`LargeMergeLands`) |

### Design (operator, 2026-10-25)

- Loom and darn handle a moved parent in Finalize itself: the conflict session resolves a conflict under the parent lock, and an overlapping merge goes on to the landing gate, whose merge fixer fixes what fails.
- A conflict or overlap in Finalize no longer reroutes, so neither recipe re-runs its Code-Review perch for it.
- Parent-Align, before Code-Review, stays the loom row that catches the bulk of the parent's drift; Finalize takes only what moved since.
- A beam's landing, the `pawl` recipe, keeps its own full treatment.

## A landing fixer's escalate goes to the arbiter first (was note `landing-escalate-arbiter`)

### Seen (2026-10-24)

A landing-stop halt at `Parent-Align` or `Finalize`, the merge fixer's `escalate` report included, goes straight to the orch: `internal/arbiterstop` marks it `LandingHalt` and skips the arbiter, and `landingshed.HandsBack` names it a hand-back.
The fixer escalates when a decision in the task collides with work the parent brought in, the one case with a real judgement in it.

### Design (operator, 2026-10-24)

- A fixer's `escalate` at a landing row goes to the arbiter first, as other stops do: the arbiter can likely settle a decision collision against the decision record and what landed, and only its hand-on reaches the orch.
- A spent cap, a failed or unusable fixer turn and an over-cap change stay as they are.
- `ArbiterSeam` has an optional `JudgesFirst func(producer string) bool`; a blocked stop at a row it marks goes to the arbiter, skipping the tackler, and only the Webster row is marked today.
  A landing fixer's escalate is one more row for the seam, not a new mechanism; the landing-halt rule row that precedes it must yield for that case.

## Docs

The orch's role text on landing halts (`Landing halts` in the orch role stencil) changes with the behaviour: an escalate reaches the orch only as the arbiter's hand-on, and a moved parent no longer reroutes.
