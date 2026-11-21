---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: in `contracts/specs/loom-discussion-spec.md:124` the opening line of `### A bug entry's record` now reads "The Discussion-Write lead's record for a `bug` entry states up front what the red-first gate later checks:".
  The rule no longer binds darn Record-Write records.
  The three bullets and the waiver paragraph are unchanged.
  The review's location line (337) was wrong; the text sits at line 124.
  The wording differs from the review's example only to keep the line under 120 characters.
  Commit `4fff7d66` made the scope change; a follow-up edit shortened the line (see Self-check, fix-affected-set).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, rubric bullet `loom-rubric-discussion-review.md:48` is unscoped — disputed, premise false: the Discussion review rubric reads only the Discussion-Write lead's record, so the exemption never reaches it.
- fix-proposal: no findings.
- fix-affected-set: LOW, line 124 was 122 characters — fixed by rewording to about 106 characters.

## Disputed

- fix-consistency LOW (rubric bullet unscoped): ground premise false.
  `contracts/stencils/loom/loom-rubric-discussion-review.md:47-48` belongs to the Discussion review, whose subject is the lead's record; the lens itself notes the gap is small in practice.

## Deferred

Nothing deferred.

## Rulings

No ruling reached this round.
