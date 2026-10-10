---
verdict: CONTINUE
rationale: "Round 2 carries BLOCKING F1 (resume probe stops a Done advisor, key done-advisor-strand-removed, open since round 1) and design MEDIUM F2 on key advisor-start-failure-notice-before-chair-exists, also open since round 1; F3 and F4 are design MEDIUM on keys first raised in round 2."
---

Verdict is `CONTINUE`.

- F1 is BLOCKING `design`, so it blocks on its own; I map it to the earlier key `done-advisor-strand-removed`, reopened in a new form (resume across a Done advisor).
- F2 is `design` MEDIUM and maps to the earlier key `advisor-start-failure-notice-before-chair-exists`, unchanged in substance (no recipient before the chair's handle exists), so it also blocks.
- F3 (unserialized senders) and F4 (chair-to-advisor edge) are `design` MEDIUM on keys first raised in round 2, `send-notices-unserialized` and `chair-to-advisor-edge-unbounded`.
  They would not block alone, but convergence needs no BLOCKING and no earlier-key gating finding, and F1 and F2 remain.
- Round 1 keys `send-notice-needs-idle-chair`, `advisor-to-parent-route-unbounded`, `still-live-undefined`, `provider-word-single-source-claims-false`, `added-after-discussion-duplicates-d6` and `decisions-relitigate-rejected-alternatives` were not raised again; the review's focus verdicts confirm D7's `Run.Send` description matches the code.
  I mark them resolved.
- The review has no `## Focus departures` section, and I relabel no finding.
- No finding cites a PATTERN entry.
