---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) sits on d16-d17-dropped-landing-notice-no-recovery, open in rounds 1 and 2 and still open in round 3, so the gating finding circles; F2 (MEDIUM, consistency) maps to d11-per-site-verdict-finish-line, is not gating-class, and no relabels were made."
---

F1 is the dropped-notice fallback.
D16 names `origin/main` against `origin/prod` as the durable baseline, D17 gates that comparison on an empty deploy key, and this hub sets the key.
A dropped landing notice therefore leaves the orch no durable signal.
The key has been open since round 1 and each round's fix has left a baseline that fails in the hub's shipped configuration, so the next fix should change the baseline, not reword it.

F2 is the remaining D11 wording ambiguity about whether every Warn line is audited.
Its class is `consistency`, so it is not gating; the fix is a local rewording the finding text already specifies.

No finding was relabeled and no focus departures were present in the review.
No entry in the PATTERN overview conflicts with the decision rule.
