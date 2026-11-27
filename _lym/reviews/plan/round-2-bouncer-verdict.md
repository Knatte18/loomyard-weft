---
verdict: CONTINUE
rationale: "Round 2 carries two BLOCKING findings, R1 (reportUpdateFrom loses the placed window's place under mode off) and R2 (card 8's contract test skips by design), and R3 is a design MEDIUM on a key open since round 1; no relabels."
---

The latest round has two BLOCKING findings, so the loop has not converged.

- R1, BLOCKING, design: a new key.
- R2, BLOCKING, consistency: a new key, citing `PATTERN-test-speed`, so it stays BLOCKING.
- R3, MEDIUM, design: maps to `placement-child-registration-and-slug-readers`, open in round 1.
  It would block convergence on its own.

The review carries no `## Focus departures` section, so there is nothing to ratify.

The review notes that the decision record asks for the placement rules under tier-1 tests while the plan puts them at `tier0`.
`PATTERN-test-speed` puts a behavior at the lowest tier its real modules run in, and the entry wins over the record.
