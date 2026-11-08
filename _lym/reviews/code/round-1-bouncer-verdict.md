---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (OS-bound subtest registered on Windows, a PATTERN-test-speed violation) and MEDIUM design F2 (zoom skip untested); no relabel applied."
---

The latest round is round 1, with no earlier ledger and no `## Focus departures` section.

- F1 is BLOCKING, class `consistency`, citing `PATTERN-test-speed`. A BLOCKING finding forces `CONTINUE` and is never downgraded.
- F2 is MEDIUM and class `design`, which is gating at MEDIUM or worse.
- F3 is LOW and class `design`; it does not gate.

No finding is relabelled.
