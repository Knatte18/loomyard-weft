---
verdict: CONTINUE
rationale: "Round 1 carries two BLOCKING findings, F1 (design: hand-written Mode, Origin and TaskHead seams in prreview_mail_integration_test.go) and F2 (decision: new uses of prkit.API and creelfake.Delivery), plus design findings F5, F6 and F7 at LOW; no finding was relabelled."
---

The loop has not converged.
The decision rule continues on any BLOCKING finding, and this round has two: F1 and F2.
Both rest on `PATTERN-test-stand-ins`: F1 cites the entry, and F2 cites the inventory the review says the entry points at.
Neither is downgraded.

F2's fix text defers it to the run's parent, since it needs a ruling on whether this task builds the missing recorded stand-ins or the two standing fakes are accepted.

The review carries no `## Focus departures` section, so there is no departure to ratify.
