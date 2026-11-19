# Card 12 — harden-rubric

**Create:**
- `contracts/stencils/harden/harden-rubric-review.md`
- `plan:contracts/stencils#HardenRubricReview` -> `var HardenRubricReview []byte`

**Edit:**
- `contracts/stencils#entries`

**Extends:**
- `.gitattributes`

**Uses:**
- `contracts/stencils#LoomRubricCodeReview`
- `contracts/stencils#TestRegistry_MatchesOnDiskTree`

**Intent:**
The `harden-rubric-review` stencil, read by the Harden-Bouncer judge and the Harden-Burler-Review row alike (D3, D12).
It is a rubric marker value in `loom-rubric-code-review.md`'s shape: a leading comment naming its readers, then the rules, using only the markers `shedadapters.ReadRubric` fills.
`HardenRubricReview` embeds it beside the loom rubrics, `entries` registers `harden-rubric-review`, and `.gitattributes` pins its line endings as the other stencils' are.

The stencil carries every rule a round and its judge act on:
- Review before fix: the review half only reviews and writes its review file.
- Log as you go.
- Deploy first after every source change, through `lym harden sandbox new`, and invoke only the `.dev-bin` lym it prints against a sandbox hub.
- Drive the real substrate rather than trace code.
- Sabotage-prove every new regression test, and re-drive each prior round's fixes.
- A sequential control and a second hub for every concurrency claim.
- For a module that drives a real LLM, one named tier-3 test at a time.
- Make every hub through `lym harden sandbox new` and none outside the sandbox root; leave zero stray state.
- Never run `lym selfreport create`.
- A finding whose fix is a feature or a refactor outside the module is deferred by the fixer, never fixed in the round.
- The judge rules CONVERGED only on a round whose review records no BLOCKING or MEDIUM finding, whose live gate is green, and whose reviewer model differs from the previous round's, read from the round's facts.
- When the open findings repeat a shape, the focus file assigns an enumeration sweep of that shape before the judge rules CIRCLING.
- The focus file derives the next round's assignment from what the last round left standing.
- A convergence verdict states what the campaign did not drive.

The stencil's static prose takes no test of its own; `TestRegistry_MatchesOnDiskTree` checks the registered file against the tree.

**ImpactSummary:** One new stencil file and its registry row; no existing stencil changes.
