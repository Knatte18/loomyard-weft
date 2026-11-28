---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "Added the closed-scope clause ahead of the stop instruction, not after line 40 (line 56 gated) as proposed: there it would sit between the stop report's way forward and \"Then create the stop marker\", so the marker step would read as part of the exception."
  - id: F3
    disposition: fixed
---

## Changes

Both `contracts/stencils/darn/darn-fragment-make-change.md` and `contracts/stencils/darn/darn-fragment-make-change-gated.md` changed, with identical wording.

- F1 (fixed differently): on a task a Code-Review round has run on, a `Decision for <slug>:` board note that says the scope is closed and files the measurements now settles the point: the writer runs none of the calls, does not stop, makes what the record asks without them and names the note in the design decisions.
  The final stop sentence now reads "With no such entry or note, run none of them and stop."
  Not made: the proposed placement, for the reason above.
- F3 (fixed): "after which you run them" is now "after which the resumed writer runs them".
- From the self-check: the stop report's closed-scope way forward now names a note titled `Decision for <slug>: <call>` that says the scope is closed and files the measurements.

Commits: `d2f0be1c8` (F1, F3) and `ad4e247dd` (F1, self-check correction).
`lym gate test ./internal/darnengine ./contracts/stencils` passed after each (208 of 208).

## Self-check

- fix-code-correctness, LOW, the stop report's way forward asked for less than the new clause requires: fixed as proposed (one finding with the two below), `ad4e247dd`.
- fix-consistency, LOW, the same gap: fixed as proposed, `ad4e247dd`.
- fix-proposal, MEDIUM, the same gap: fixed as proposed, `ad4e247dd`.
- fix-affected-set: no findings.

The fix-proposal fork also noted, without raising a finding, that it did not establish how the later gates treat a measurements-only task settled by such a note with no commit on the branch.
I did not check it either.

## Disputed

## Deferred

## Rulings
