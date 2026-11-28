---
dispositions:
  - id: F16
    disposition: fixed-differently
    reason: "Made every deletion and the replacement passage the fix names, but did not keep the batten refusal's `lym batten cleanup --apply` clause. The jacquard check runs only for a start with no status file, and refuseAtStart (internal/battencli/startguard.go:530) refuses a done slug that no clear admits before that check, so the clause could never show. The bullet now says a slug holding a done run meets doneSlugRefusal (internal/battencli/arm.go:945) first, and the passage names the refused-Sketch draft task that refusal guards."
  - id: F17
    disposition: fixed
---

## Changes

All edits are in `_lym/discussion/decision-record.md`.

- F16, fixed differently: in D5, the motif-run clear passage is replaced with three sentences.
  A finished motif run turns its entry into a Sketch (`finishesAsSketch`), and a Sketch the board refuses leaves a draft task over the done run (`sketchFinish`).
  Only a raddle run takes a Sketch (`checkDiscussedPromotion`).
  So jacquard builds the parts raddle files and other discussed tasks, and batten gains no motif-run clear.
  D5's Bound no longer mentions reading the done run.
  The criterion that a jacquard entry starts after a done motif run is deleted, and so is the Constraints Fact about a motif run's task branch.
  I did not keep the batten bullet's `lym batten cleanup --apply` clause, which the fix said to keep.
  The bullet now says a slug holding a done run never reaches the jacquard check: `doneSlugRefusal` refuses the start first and names `lym batten cleanup --apply`.
  Evidence: `refuseAtStart` (`internal/battencli/startguard.go:530`) clears a done status only when `clearsMotifRun` admits it, and that stays raddle-only.
- F17, fixed: its sentence went out with F16's passage, as its fix says, and nothing else changed.

## Self-check

- fix-consistency: no findings.
- fix-claims, LOW: the Sketch sentence was unconditional, though the board can refuse a Sketch. Fixed: the passage now names the draft task `sketchFinish` leaves.
- fix-proposal, MEDIUM: the kept cleanup clause can never show, since the done-run guard refuses first. Fixed with the lens's second option: the bullet names `doneSlugRefusal` as the refusal a done slug meets.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
