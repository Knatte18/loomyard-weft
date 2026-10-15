# Fixer report — plan, round 1

## Changes

- **F1** (LOW, consistency), fixed as proposed in `_lyx/plan/04-judge-parts.md`'s `TestDirective_RendersEveryValueVerbatim` bullet.
  The want "returned no report.\n6. Run `lyx check --final` again" moved from the "without a writer fork" row into the shared wants.
  A new shared want, "forks are outstanding is allowed.\n4. Judge every finding", pins the leading edge.
  Both judge parts open with "4. Judge every finding" (the critical part as today's step 4, the plain part as `main`'s step 4), and both end with "Name a fork that returned no report.", so the two shared wants hold at both hosts.
  The placement sentence now attaches to the shared wants and says "at both hosts".
  No target or verify command changed.

## Disputed

None.

## Deferred

None.
