---
verdict: CONTINUE
rationale: "Round 1 carries one BLOCKING finding (F1, card 11 misses the TestNames ShipsExpectedEntries pin, so it fails its own card gate); no finding is design-class, but any BLOCKING finding forces CONTINUE."
---

Round 1 has 11 findings: 1 BLOCKING, 4 MEDIUM, 5 LOW, 1 NIT.
None is `design` class, so no finding is gating by class; F1 is BLOCKING and so forces `CONTINUE` on its own.

No finding was relabelled, and the review carries no `## Focus departures` section to ratify or reject.
No finding cites a PATTERN entry as a violation except F8, which cites `PATTERN-test-economy` at LOW; that is an ordinary severity and stays as the review set it.

F2, F3, F4 and F5 are MEDIUM but `scope` or `consistency` class, so they do not gate.
They are still fixed or disputed with evidence under `PATTERN-review-round`.

This is the first round, so there is no earlier ledger and no recurring or earlier-open key.
