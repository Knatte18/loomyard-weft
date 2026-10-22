# Discussion settles a fact the design rests on; reviewers flag a deferral

A factual question a design rests on is now researched and answered in Discussion, never handed to Plan or an implementer with a halt-on-discovery rule.
The `ubuntu-2404-support` run showed the cost of the old behaviour: Plan gave a card the tmux feature audit with a rule to halt on a new above-floor form, the implementer found `allow-set-title` in minutes, and the halt cost a failed batch, a recovery, a tackler and a fresh Plan review.

## What changes

- `contracts/stencils/loom/loom-template-discussion.md` — Step 2 gains a rule: a question that asks a fact, not a choice (tool or dependency versions, upstream behaviour, what the code does today), is researched from the code and from upstream changelogs, manuals and docs at the versions that matter.
  Its answer goes into the decision record's Decisions or Constraints with its evidence named by source and version.
  Deferring it, or handing downstream a rule to halt on discovering it, is forbidden.
  The chair stencil includes this template whole, so the seat-table lead gets the rule too.
- `contracts/stencils/loom/loom-rubric-discussion-review.md` — a new `## Also flag` bullet: a Decision, Constraint, risk or acceptance criterion that leaves such a fact to be found out later, or a halt-on-discovery rule handed downstream, is a finding.
- `contracts/stencils/loom/loom-rubric-plan-review.md` — a new `## Also flag` bullet: a card that leaves a fact the design depends on to be found out while it runs, or carries a halt-on-discovery rule, is a finding, fixed by researching the fact into the plan, or into the decision record through the parent when the answer changes the design.

## Design decisions

- Step 2's existing fence (no exact signatures, `file:line` citations, interface shapes, dependency lists or exhaustive existing-pattern research) stays.
  The new rule states that the fence excludes plan-level detail, never a fact the design rests on, so a writer cannot read the fence as licence to defer.
  The Discussion-Review bullet likewise exempts a fact the writer was told not to gather, which keeps the rubric's writer/reviewer symmetry.
- The Plan writer, advisor and darn stencils are unchanged.
  The Plan-Review rubric catches a deferring card, the chair owns the record that carries the research, and a darn writer implements its own design, so it hands no fact to another session.
- No test is added: the change is stencil prose, which `PATTERN-test-economy` keeps out of assertions.
  Neither rubric gains a marker, so the rubric marker tests in `contracts/stencils` cover them unchanged.
