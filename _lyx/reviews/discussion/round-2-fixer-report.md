# Fixer report — discussion, round 2

The one finding was validated against D2's pass rule and fixed.
`lyx loom validate-discussion` passes on the result.

## Fixed

- **F1** (MEDIUM): confirmed against D2's pass rule that a failing change passes once the head is at most the larger of the base size and the line.
  So the smallest passing cut is `head - max(base, line)`, and naming the cut to the line overstates it whenever the base sits above the line.
  The growth-only rule is kept, since D2 and D3 exist to hold a branch to its own growth on top of main's content.
  D2's finding now names the bytes to cut to the larger of the base size and the line, and says that with the base above the line this is the branch's own growth.
  D5's Rationale is restated: the merge base after a Publish merge-in is the merged-in parent, whose overview passed the hard cap, so the named cut is at least what the cap needs, and the fixer cuts that much.
  The headroom function's acceptance rows split the failing-growth case into growth from under the line, which names the cut to the line, and growth from a base above the line, which names the cut back to the base size.
  The merge-in acceptance row now fails a branch that grew the overview on top of the parent's content, naming the bytes of its own growth.
  support-log.md gains the rejected alternative of failing any growth until the head is at or under the line.

## Disputed

None.

## Deferred

None.
