---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed Blast radius phrase and Door explanation break the section format: the final-summary spec (loom-final-summary-spec.md:44, :49-50) allows exactly one-way or two-way for Door and one word for Blast radius. Door is now one-way and Blast radius is the one word landing; the reasons (suppressed notice under a recorded pull request, dropped member key, orch upkeep losing its prompt) sit in Design decisions."
---

# Round 1 fixer-report

## Changes

- F1, `_lym/landing/summary.md`: `## Merge danger` now reads `**Door:** one-way` and `**Blast radius:** landing`.
  A new last bullet under `## Design decisions` states why the door is one-way and what the blast radius covers.
  The file is under `_lym`, so it is edited in the working tree and not committed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, BLOCKING: Door and Blast radius lines break the one-word format; fixed by the change above.
- fix-consistency, MEDIUM: the widened radius cannot be written as proposed; fixed by `landing` and the Design decisions bullet.
- fix-proposal, BLOCKING: same format failure as fix-consistency; fixed, merged as a duplicate.
- fix-proposal, LOW: door judgment rests on the rubric's setting-meaning rule, not revert-safety; kept one-way, stated in the new bullet.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
