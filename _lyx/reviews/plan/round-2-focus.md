---
round: 2
focus:
  - "Merge-In fixer cards (the card that adds the fixer to Merge-In and the cards that define fixer attempts and the fixer run): settle whether the point where the fixer's `AlignFrom` fork point is recorded survives an error stop after the merge and the `landing` gate but before align runs, so that a resumed Merge-In still spawns the fixer over parent commits it never aligned."
  - "Fixer attempt and run cards together with the Merge-In and Finalize fixer cards: settle what state a resumed entry reads when the last turn ended by timeout, no-report, dirty or pushed, leaving `Spent` at the cap with no spent or over-cap Halt note, and whether any resume spawns a turn past `merge_fix_attempts` or `finalize_fix_attempts`."
  - "Fixer attempt, Merge-In and Finalize cards and the HandsBack card: settle whether every halt whose way forward grants a fresh fixer cap, including the undecodable-attempt-record halt, is marked so that only the orch's `lyx loom resume` lifts it and no tackler or arbiter can take the step."
---

Focus targets the design-class questions the gate turns on.
Entries name where to look and which question to settle, and leave severity to the reviewer.
