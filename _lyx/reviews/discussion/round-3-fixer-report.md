# Round 3 fixer report

All six findings are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` needed no change.

## Fixed

- **F1** The Goal now words the plan gate's guarantee to D13's scan scope: a `**Verify:**` value, the verify section or a fenced block for both matches, and an inline code span for the sibling path.
  D13's Bound now lists a spelling in an inline code span or prose as a miss, and the Open risks bullet follows.
- **F2** D13's Bound no longer says the check ignores absolute paths and `..` escapes; it says such a path is refused when it carries a name ending in the weft suffix.
- **F3** Constraints gained a `PATTERN-fabric-vocabulary` entry: the finding, the specs and the template say "fabric repo", and tests outside the fabric side drive the check through the matcher seam with weft-named inputs built by the fabric side's exported helpers.
- **F4** D13 no longer claims the audit's exact-path branch is subsumed; the audit keeps it, and the shared rule is the three location-free patterns.
- **F5** D11 states that a profile line's findings per card is taken over the cards of only the runs that contribute findings, and the line states how many runs those are; the acceptance criterion follows.
- **F6** D5 words its resume claim to the estimate: never a multi-card batch whose estimate exceeds `budget`.

Prior rounds' fixes were checked and none regressed.

## Deferred

None.
