# Settle researchable facts in Discussion; reviewers flag deferrals

A fact the design rests on is now researched and recorded in Discussion.
Before this change, a task could hand such a question to Plan, or to an implementer with a rule to halt on discovery.
In run `ubuntu-2404-support`, an implementer fork found an above-floor tmux feature that way, and the halt cost a failed batch, a recovery, an orch decision and a re-plan.
The same research done in Discussion takes minutes.

The change is stencil prose only, under `contracts/stencils/loom/`:

- `loom-template-discussion.md`: the writer's exploration step gains a rule.
  A factual question the design rests on (a tool or dependency version, upstream behaviour, what the code does today) is researched from the code and from upstream changelogs, manuals and docs at the versions that matter.
  Its answer goes into the decision record's Decisions or Constraints, with the evidence named by source and version.
  The writer never leaves such a fact for Plan or an implementer, and never hands downstream a halt-on-discovery rule.
  A question that asks a choice rather than a fact still goes to the interview.
- `loom-rubric-discussion-review.md`: `## Also flag` gains a bullet.
  A Decision, Constraint, risk or acceptance criterion that leaves such a fact to be found out later, or hands downstream a halt-on-discovery rule, is a finding.
  The fix is the research done and recorded with its evidence.
- `loom-rubric-plan-review.md`: `## Also flag` gains a bullet.
  A card that carries a research step the design depends on, or a rule to halt when its work discovers such a fact, is a finding.
  The finding names the fact.
  The fix is the fact researched and recorded in the plan, or in the decision record through the parent when the answer changes the design.

## Design decisions

- The new writer rule widens the exploration step only to facts the design rests on.
  The step's exclusions stay: no exact signatures, `file:line` citations, interface shapes, dependency lists or exhaustive existing-pattern research.
  The rule says so in a sentence of its own, so "no exhaustive research" cannot be read as licence to defer a fact.
- The Discussion rubric's bullet exempts a fact the writer was told not to gather, so the writer/reviewer symmetry the rubric keeps still holds.
- The Plan writer stencil, the advisor stencil, the darn stencils and rubric, all Go code and the discussion spec are untouched.
- No marker and no conditional is added to any of the three files, so the existing rubric marker tests are unaffected.

## To check by hand

- Nothing was run against a live Discussion or Plan session.
  Whether a writer now researches a version-dependent fact, and whether the two reviewers raise the new findings, shows only on a real run.
