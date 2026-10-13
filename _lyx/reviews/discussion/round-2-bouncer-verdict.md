---
verdict: CONTINUE
rationale: "F1 (design, MEDIUM) reopens the key d16-d17-dropped-landing-notice-no-recovery, which the facts list as open in round 1; the D17 fallback still has no durable baseline, so the gating rule gives CONTINUE."
---

Round 2 carries two MEDIUM findings and no BLOCKING one.

- F1 is class `design` and maps to `d16-d17-dropped-landing-notice-no-recovery`, a key open in round 1.
  The round 1 fix added a next-wake fallback, but it compares against landings the orch handled, which nothing records across a refresh, compact or handoff.
  That is a gating finding on a key open in an earlier round, so the round does not converge.
- F2 is class `consistency` and maps to `d11-per-site-verdict-finish-line`.
  It is not gating-class, so it does not block alone, but its fix is carried forward.
- No finding was relabelled.
  The review has no `## Focus departures` section.
- The review rejected the D13 import-cycle and D5 dedupe concerns with evidence, so those keys are resolved.
  `record-forensic-narrative-relocation` was not raised again and is resolved.
