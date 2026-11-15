---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (pure shedcheck and shedbuild.Check kept in production under the allowlist, against PATTERN-test-only-code) and gating-class MEDIUM findings F3 and F4; F2 is a MEDIUM decision deferred to the parent. No finding was relabeled."
---

The decision rule gives `CONTINUE` while the latest round has a BLOCKING finding or a gating-class (`design`) finding at MEDIUM or worse.

- F1 is BLOCKING and `design`: it cites a violation of `PATTERN-test-only-code`, so it is gating and stays BLOCKING.
- F3 and F4 are `design` at MEDIUM.
- F2 is `decision` at MEDIUM. It is not gating by class, and the review defers it to the parent.
- The remaining findings are LOW or NIT, or `consistency` class.

No finding's class or severity is departed from.
The review has no `## Focus departures` section.
