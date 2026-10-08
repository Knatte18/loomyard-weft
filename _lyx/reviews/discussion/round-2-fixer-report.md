# Round 2 fixer report — discussion

## Fixed

- **F1** (MEDIUM, design): D5 now makes a lens eligible for exclusion only if the latest round ran it; Go drops, with a warning naming them, `exclude_lenses` entries for lenses the latest round did not run, so the first round under a switched-in fan runs it whole.
  The judge-stencil sentence in D5 and D5's Bound say the same, and the exclusion Acceptance criterion gains the case, including a switch from solo or from another fan.
- **F2** (MEDIUM, design): the discussion fan is now `[generic, goal-scope, pattern-fit, scenarios, leanness]`.
  `leanness` names the Discussion-Review rubric's relocation-and-exclusion item and its completeness-before-leanness test.
  `edge-cases` and `acceptance` merged into `scenarios`, which names the attack-surface item and the discussion stencil's must-cover-scenarios rule instead of adding a one-criterion-per-decision rule, so the fan keeps five lenses.
  D3's Rationale now says which lenses come from the rubrics and which from the writer's interview categories; support-log Interview item 5 and Q4 match.
- **F3** (LOW, consistency): D3 now limits the no-lens-edit claim to a change in an item's rule, and states that a renamed or removed rubric item needs an edit to the template lens naming it, which a seeded hub picks up only when its `burler.yaml` does not define that lens's name (D4).

## Disputed

None.

## Deferred

None.
