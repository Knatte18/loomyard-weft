# Fixer-report, code review round 1

## Changes

- F2 (LOW), fixed as proposed: `contracts/stencils/orch/orch-template-role.md:188` now reads "and lym adds no instruction file to a repository". Commit `35fe8ff6`.
- F3 (NIT), fixed as proposed: `pattern/PATTERN-parent-directive.md:13` now reads "the only statement of the edit directive's rules". Commit `30b4eaa0`.
- Self-check, fix-proposal LOW 1, fixed with my own change: `pattern/PATTERN-parent-directive.md:15` now reads "A test enforces the edit rule.", since the test guards only the edit rule.
- Self-check, fix-proposal LOW 2, fixed with my own change: the `//testtiming:keep` reason at `contracts/stencils/parentdirective_test.go:141` no longer quotes the old PATTERN wording.

F1 (MEDIUM, `decision`) went to the parent, who closed it with decision D2: the drift baseline stencil gets the pointer replacement only, so no change was made. It is left out of this report's dispositions.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings; it noted the stale test comment, which fix-proposal LOW 2 covers.
- fix-proposal, LOW: PATTERN line 15 "A test enforces the rule" overstated after F3; fixed with my own change.
- fix-proposal, LOW: stale `//testtiming:keep` comment at `parentdirective_test.go:141`; fixed with my own change.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
