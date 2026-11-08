---
verdict: CIRCLING
rationale: "Round 3 carries BLOCKING F1 (design) on goto-raddle-write-no-rearm, plus MEDIUM design F2 on rearm-keeps-generation-record, F3 on split-perch-auto-accept-unbounded and F4 on d6-admission-unbounded, all open in earlier rounds; the last two have recurred since round 1. No focus departures to ratify."
---

Mapping of round 3 findings to ledger keys:

- F1 (BLOCKING, design) maps to `goto-raddle-write-no-rearm`, open since round 2.
  The round 2 fix rested on `freshRaddleEntry` matching a goto entry; F1 shows it does not.
- F2 (MEDIUM, design) maps to `rearm-keeps-generation-record`, open since round 2.
- F3 (MEDIUM, design) maps to `split-perch-auto-accept-unbounded`, open since round 1: the soft-cap bound still misstates what the rule lets through.
- F4 (MEDIUM, design) maps to `d6-admission-unbounded`, open since round 1: the bound is still not the true one.
- F5 (LOW, design) is a new key, `d6-order-check-underdetermined`.
- F6 (LOW, design) maps to `filing-mark-after-board-write`, open since round 2, now narrowed to the store-done retry case.

No finding is relabeled.
The review lists no focus departures.
The gating findings sit on keys already open in earlier rounds, so the rule gives CIRCLING.
