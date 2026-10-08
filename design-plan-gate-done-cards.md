# The plan gate fails done cards after a goto back to Plan-Review, and rebaseline refuses follow-up cards on an older webster record

`plan-gate-done-cards` · Note · bug, loom, webster

Imported from [issue #463](https://github.com/Knatte18/loomyard/issues/463).

Seen 2026-10-08 in `shuttle-send`, following #462: the hub moved a run blocked at Publish back with `lyx loom goto --to Plan-Bouncer`, so Plan-Review would raise an uncarried parent decision against the plan.

- The plan gate validates every card against HEAD, so cards 1-10, which Webster had already run, all failed (`create-already-exists`, `glyph-not-found`, `path-missing`, `caller-uncovered`); the fixer could clear them only by retiring or rewriting done cards.
- The fixer added follow-up cards 11 and 12 with Card Index lines, but `lyx webster rebaseline --card 11 --card 12` refused: "00-overview.md changed and this run recorded no overview frame", because the run's webster record predates overview-frame tracking. Its only way forward, `lyx webster reset --to start`, would re-run all ten done cards.
- `goto` from Plan-Bouncer cannot reach Webster, so there is no route past the plan gate once the plan holds done cards.

The hub restored the recorded plan and will fix the defect forward on the task branch at Publish.

Expected: a plan gate after Webster began judges only cards after the last begun batch (done cards are history, not targets), and a webster record without an overview frame accepts a Card-Index-only change, or records the frame lazily.
