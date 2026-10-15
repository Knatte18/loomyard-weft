---
round: 1
ledger:
  - key: shared-golden-under-package-testdata
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: idless-key-fallback-collides-across-transcripts
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: sessionusage-row-pins-behavior-change
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card5-resignatures-without-arrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four consistency findings against the plan.
F1 (BLOCKING): the `reads.json` golden is shared by `claudeengine` and `tokencount` tests, so it belongs in `internal/testkit/shuttlefake`.
F2 (MEDIUM): the `line:<n>` message-key fallback collides across transcripts.
F3 (MEDIUM): a `TestSessionUsage` row expects a behavior change D5 rules out.
F4 (LOW): card 5's `messageCost` and `run` re-signatures carry no arrow, with no admissible target change.
