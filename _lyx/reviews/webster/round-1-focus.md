---
round: 1
focus:
  - "discussionparser `CheckRelevant` (card 3): settle whether it matches the format pinned in the discussion spec (card 1), and whether an absent list passes while a present malformed one fails."
  - "`ValidateLoomDiscussion` in loomshed (card 4): settle whether the loom discussion gate and `lyx loom validate-discussion` both reach the check through this one function, and that no other caller path was left running the old check."
  - "Stencil edits (cards 5, 6, 7): read each against its card's Intent; settle whether any rule the card did not ask to change was dropped, weakened or contradicted, and that the Plan stencil treats relevant.md as a head start, never scope."
  - "Test economy: settle whether the new tests duplicate existing coverage, whether any behavior the diff adds (accessor, check, gate wiring, twill relaunch file set) has no test, and whether the final wiring-test fix adds coverage beyond the existing wiring test."
  - "Per-card mechanical checks: settle whether each card's own groups ran their type-specific checks against every target, not only the first label."
---

Range: commits `b97577d98` through `c5047ed3e`, the eight card commits plus the wiring-test fix, against the merge base with the parent branch.
No rework round exists, so step 4 of the range definition applies.
