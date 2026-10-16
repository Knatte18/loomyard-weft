---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) sits on key d16-d17-dropped-landing-notice-no-recovery, open in rounds 1 and 2 and still open in round 3, so a gating finding persists on a recurring key; F2 (MEDIUM, consistency) maps to d11-per-site-verdict-finish-line, recurring but non-gating."
---

F1 maps to `d16-d17-dropped-landing-notice-no-recovery`.
Rounds 1 and 2 raised the same gap: the orch has no durable signal for a dropped landing notice.
Round 3 shows the round-2 fix gave a baseline (`origin/main` versus `origin/prod`) that cannot work where the hub's deploy key is set, and F1's fix, a board-done check independent of the deploy key, is a design change.

F2 maps to `d11-per-site-verdict-finish-line`.
It is a wording inconsistency of class `consistency`, so it is not gating, and its fix is mechanical.

No finding was relabelled, and no focus departures were present in the review.
