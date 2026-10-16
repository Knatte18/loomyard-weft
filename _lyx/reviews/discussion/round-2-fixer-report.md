# Round 2 fixer report — discussion

## Changes

- **F1** (fixed as proposed): D5's Bound now says the `## verify:` section is also the chained command Publish and Finalize run after a parent merge, so a compiled-only tier goes unrun at landing too.
  It splits the bounds: the Guard classification bounds who writes an entry, the card and batch gates bound the skip inside webster only, before the parent merge, and at landing only `publish_verify` bounds it, running what landing config names.
- **F2** (fixed as proposed): D1's Decision adds that the rework stencil states a finding overturning no Decision is not a late scope addition under the plan stencil's route and is planned directly, licensing its card's scope, as the rework stencil's own exception, which its Step 1 allows.
  The rework clause of Acceptance criteria gains the matching clause.
- **F3** (fixed as proposed): D6's implementer bullet keeps the `tmux`/`llm` compile-only sentence (`lyx gate test --tags <tag> <packages> -- -run '^$'`) as a gate-describing sentence beside the guard sentence, with the list's compile-only rule adding tiers beyond those two.
  The implementer clause leaves the first Open risks bullet, and the Acceptance criterion's Go-command exception names that sentence.
- **F4** (fixed as proposed, one addition): D6's plan bullet says the requirement to run each targeted package's tests, its hermetic build-tagged tests included, stays as a language-neutral rule with its `-tags integration` example dropped.
  The clause stays, so no Open risks addition is needed.
  D6's rubric bullet gains the same clause, because `loom-rubric-plan-review.md:93` carries it beside the plan stencil's and the two would otherwise disagree.
- `support-log.md`'s Review rounds gains the round 2 line.

## Disputed

None.

## Deferred

None.
