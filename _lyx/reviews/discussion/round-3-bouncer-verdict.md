---
verdict: CIRCLING
rationale: "Gating design findings stay flat at 3 MEDIUM in rounds 2 and 3, and round 3's F1, F2 and F3 reopen the round-2 keys d4-budget-test-off-by-one, circling-continue-hits-shed-block and d8-bound-reader-undecided."
---

The latest review carries three `design` findings at MEDIUM: F1, F2 and F3.
No finding was relabelled, and no `## Focus departures` section was present.
The facts table lists no recurring keys because ledgers are matched by key, but the mapping is direct:

- F1 (D4 budget comparison is not Shed's) is `d4-budget-test-off-by-one`, open since round 2.
- F2 (failed-seam path still reaches Shed's block) is `circling-continue-hits-shed-block`, open since round 2.
- F3 (D8's bound depends on readers no Decision provides) is `d8-bound-reader-undecided`, open since round 2.

Gating counts went 2, 3, 3 over rounds 1 to 3, flat across the two latest fresh rounds, and all three gating findings sit on reopened keys.
The fixes between rounds 2 and 3 restated the same points without settling the boundary, the seam qualification or the reader.
No PATTERN entry is cited by any finding, so none gets special weight.
The next round should settle the three points at their root rather than patch the wording again.
