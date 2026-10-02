---
round: 1
ledger:
  - key: passoncap-bound-vs-reset-on-pass
    rounds: [1]
    status: resolved
  - key: passed-ignores-unreached-check-entry
    rounds: [1]
    status: resolved
  - key: acceptance-construction-syntax-only-unachievable
    rounds: [1]
    status: resolved
  - key: refusal-spec-reachability-left-open
    rounds: [1]
    status: resolved
  - key: attach-paragraph-carries-parent-review-design
    rounds: [1]
    status: resolved
  - key: decision1-nil-test-rewrite-clause
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so there was no prior ledger to carry forward.
The review approved with six findings, and all six were fixed in `decision-record.md` before judging.

- `passoncap-bound-vs-reset-on-pass` (MEDIUM): Decision 4 claimed an at-most-`Attempts` bound that reset-on-pass breaks.
  It now bounds only a capped entry, and points a request that must not repeat to an idempotent closure.
- `passed-ignores-unreached-check-entry` (MEDIUM): Decision 6 let a final arrival that stopped at a failing `PassOnCap` entry pass with a check entry never run.
  `Passed` now requires every non-off check entry to have passed, and Decision 2 states that `finalize` never re-prompts.
- `acceptance-construction-syntax-only-unachievable` (MEDIUM): the existing tests rely on the removed 0-means-3 default and the removed keys.
  The criterion now names the explicit `Attempts` of 3 and the replaced shedrecipe tests.
- `refusal-spec-reachability-left-open` (LOW): settled as wiring guards, because the recipes are embedded.
- `attach-paragraph-carries-parent-review-design` (LOW): the review-gate closure design was dropped, and the per-run limit was kept.
- `decision1-nil-test-rewrite-clause` (NIT): dropped.
