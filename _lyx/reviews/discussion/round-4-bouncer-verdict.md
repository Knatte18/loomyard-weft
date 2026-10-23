---
verdict: CONVERGED
rationale: "Round 4 has no BLOCKING finding and its one gating-class finding at MEDIUM or worse, F1 (design, Merge-In budget-0 cap baseline), sits on a key first raised in round 4, so Go carries it into Open risks; F3 and F4 map to earlier keys but are LOW consistency, not gating, and F2 is MEDIUM consistency."
---

Round 4 carries five findings: F1 (design, MEDIUM), F2 (consistency, MEDIUM), F3 (consistency, LOW), F4 (consistency, LOW) and F5 (decision, LOW).

- F1 is the only gating-class finding at MEDIUM or worse.
  It maps to the new key `merge-in-budget0-cap-baseline-precedes-merge`, absent from every earlier ledger, so it does not block convergence.
- F3 maps to `d13-deploy-bound-false-at-zero-review-budget` and F4 to `finalize-lock-other-release-arms`.
  Both are now LOW and class consistency, wording corrections only, so neither is gating.
- F2 and F5 map to new keys and are non-gating.
- No finding was relabeled.
- The round has no `## Focus departures` section.
- No PATTERN conflict arises; the declared `PATTERN-parent-lock` and `PATTERN-recipe-owns-failures` changes are not cited by any finding.
