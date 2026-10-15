---
verdict: CONTINUE
rationale: "F1 is BLOCKING (design) and maps to d7-forkaudit-dependency-no-fallback, a key open since round 1; F2 (d3-bound-omits-trace-write) and F3 (sandbox-list-vs-unchanged-bytes) are local consistency corrections; no relabels."
---

The latest round carries a BLOCKING finding, F1, so the decision rule gives `CONTINUE`.
F1 also maps to a key in the facts' open-in-earlier-round list, so the key is unchanged and not first raised this round.
F2 and F3 map to earlier keys too, but both are consistency-class and below the gating class.
`start-here-confound-in-open-risks` has no finding in round 2 and is marked resolved.
No finding is relabeled, and the review has no `## Focus departures` section.
