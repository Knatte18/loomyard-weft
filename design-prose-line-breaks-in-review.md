# Implementers hard-wrap comments and reviews re-find it every round

`prose-line-breaks-in-review` · Note · enhancement, webster

Webster implementers keep writing prose against `scribe:prose`'s line-break rule, and the Webster-Bouncer re-finds it round after round.
Seen in `board-labels-and-skills` (closed issues #372 and #373, filed by the removed Go anomaly reporter): `hard-wrapped-go-comments` stayed open in rounds 1, 2 and 4, and `independent-clauses-on-one-line` in rounds 2–4.
Each round spends a Burler fix and a Bouncer judgment on formatting the tooling could catch.

Options to settle in Discussion:

1. A guard test or `lyx` lint that flags fixed-column-wrapped comment blocks in changed Go files, run in the card gate, so the implementer fixes it before review.
2. The implementer stencil loads `scribe:prose` (or inlines its line-break rule), if it does not already.

Check first whether the stencils from `lyx-skills-and-stencils` already fixed it for new runs.
