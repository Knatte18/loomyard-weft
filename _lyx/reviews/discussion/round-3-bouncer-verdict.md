---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) sits on d16-d17-dropped-landing-notice-no-recovery, open in rounds 1 and 2 and still open in round 3, so a gating finding persists on a recurring key; F2 (MEDIUM, consistency) is on d11-per-site-verdict-finish-line and is not gating; no relabels and no focus departures."
---

F1 is the dropped-notice fallback.
Rounds 1 and 2 each tried a durable baseline, and round 3 shows `origin/prod` cannot carry it on a hub whose deploy key is set.
The finding asks for a board-done check that does not depend on the deploy key, so the fix changes a decision and needs a fresh review.

F2 is the third pass on D11's audit reach.
It is a wording correction whose text the finding already fixes.
It is `consistency` class, so it does not gate.

The review has no `## Focus departures` section.
