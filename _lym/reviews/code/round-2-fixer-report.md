# Round 2 fixer-report

## Changes

- F1 (LOW, consistency): fixed as proposed, commit `8ce1f333`.
  `crucible/review-prompt-template.md` lines 122, 129, 132 and 203 now locate live tests by `tiergate.Require(t, 2)` and `tiergate.Require(t, 3)`, found with grep, and a new live test opens with that call instead of carrying a build tag.
  The same stale wording stood in `crucible/README.md:201`, which the review did not name; it is changed the same way.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

No fork returned without a report.

## Disputed

None.

## Deferred

None.
