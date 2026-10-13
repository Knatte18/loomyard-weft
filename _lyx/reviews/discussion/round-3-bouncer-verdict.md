---
verdict: CONTINUE
rationale: "Round 3 carries BLOCKING F1 (D10 Warn-once-per-session departs from the internal/logger Level policy, consistency class, key migration-warn-recurs-each-open-and-attach, open since round 2) plus gating-class MEDIUM F2 and F3 on keys first raised in round 3; F1 is not design-class so it is not circling, but any BLOCKING finding forces CONTINUE."
---

No Focus departures section appears in the round 3 review, so there is nothing to ratify or reject.
No finding was relabeled.

F1 is BLOCKING and cites `PATTERN-log-levels` through the Constraints line, so it stays gating-severity; its class is `consistency`, so it cannot make the verdict `CIRCLING` even though it maps to a key open since round 2.
F2 (the strands' window resolved through fallbacks) and F3 (attach selection moving other attached clients) are `design` at MEDIUM on keys first raised in round 3.
They would be carried into `## Open risks` if the loop converged, but F1 blocks that.
F4 and F5 are LOW and do not affect the verdict.

The round 2 attach-selection finding is resolved: the review states the first focus question settles in the design's favour.
