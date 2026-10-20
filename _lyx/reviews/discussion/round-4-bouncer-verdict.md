---
verdict: CONVERGED
rationale: "Round 4 has no BLOCKING finding; its only MEDIUM design finding, F1, sits on a key first raised this round (d3-goto-after-halt-grants-budget), so Go carries it into Open risks; F2 and F3 are LOW, and every key open in round 3 was ratified closed by round 4's focus verdicts or not raised again."
---

Round 4 carries three findings: F1 MEDIUM design, F2 LOW design, F3 LOW design.

F1 maps to the new key `d3-goto-after-halt-grants-budget`.
It does not map to the round 1 key `d3-nothing-else-grants-budget`: that finding concerned other grants, and F1 concerns the `budget_capped` mechanism the later rounds added.
A gating finding on a key first raised in the closing round does not block convergence.

F2 maps to the new key `unharvested-verdict-read-as-older-binary`, LOW.
F3 maps to `code-identity-includes-run-records`, which stays open at LOW; LOW is below the MEDIUM threshold for a gating block, so no circling.

The round 4 focus verdicts close `amendment-inherits-spent-count`, `code-base-reads-differ-from-identity` and `document-base-predates-artifact`.
`merge-in-test-misreads-commits`, `patch-id-skip-unbounded` and `support-log-stale-against-round-2` are not raised again and are marked resolved.

No relabels, no focus departures section in the review.
