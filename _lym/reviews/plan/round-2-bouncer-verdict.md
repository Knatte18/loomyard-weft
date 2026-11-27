---
verdict: CONTINUE
rationale: "Round 2 carries a BLOCKING design finding, F15 (the plan does not show the required net shrink of fabric test code and leaves it to a halt at card 15), plus F16 (MEDIUM, the open remainder of the read-failure contract). No finding relabelled; the review has no Focus departures section. Its Constraint conflict note on PATTERN-test-only-code remains open: the entry is not a declared change and so outranks the added decision entry the reviewer relies on."
---

The loop has not converged.
F15 is BLOCKING, so the decision rule gives `CONTINUE`.

Round 1's findings are resolved apart from the read-failure contract, which F16 carries at MEDIUM.

## PATTERN-test-only-code

The review raises no finding here, and a judge raises none of its own.
It reports that cards 13 to 15 add functions only tests reach,
that the added entry "A chain part may leave production code that only a named later part first calls" licenses them for this run,
and that no card updates `pattern/PATTERN-test-only-code.md` and no PATTERN change is declared for it.
Where an entry and anything else in the judge's instructions conflict, the entry wins,
so the parent may want to declare the change or have the entry's background name this temporary allowlist kind.
