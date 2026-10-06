---
verdict: CIRCLING
rationale: "F2 (design, MEDIUM) sits on scenario-steps-isolation-bound, open in rounds 3 and 4 and still open in round 5, and F1 (design, MEDIUM) is a second gating finding; no relabels."
---

Round 5 carries two design findings at MEDIUM: F1 and F2.

F2 maps to `scenario-steps-isolation-bound`.
The facts list that key as open in rounds 3 and 4, and F2 still finds the merged-step isolation bound unsound, now because its restore clause defines no observable state and no check.
The wording changed each round, but the question is the same: what a merged step may leave behind for the next one.
That is a gating finding on a recurring key, so the verdict is `CIRCLING`.

F1 (`scenario-merge-tier-bound`) is new and gating on its own, so the loop would be `CONTINUE` even without F2.

Non-gating findings, all open:

- F3 (decision, MEDIUM) on `scenario-merge-per-family-measurement`, open since round 3.
- F4 (consistency, LOW) on `scenario-alone-rerun-mapping`, open since round 4.
- F5 (consistency, LOW), new key `scenario-serial-vs-fold-parallel`.
- F6 (consistency, LOW) on `scenario-decision-form-and-coverage`, open since round 3.

The round-4 keys `coverage-figure-directory-filter`, `scenario-parallel-stop-on-failure` and `support-log-leftovers` were not raised again and are resolved.

Focus departures: the review states none, so there is nothing to ratify or reject.
No finding is relabelled; none cites a PATTERN entry.
