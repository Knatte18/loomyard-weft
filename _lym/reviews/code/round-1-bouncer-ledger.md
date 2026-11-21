---
round: 1
ledger:
  - key: command-failure-only-note-header-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: tail-backtick-fence-breaks-note-block
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: flaky-gate-fixture-same-log-both-runs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rerun-pass-log-calls-command-failure-flaky
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: classification-rule-worded-wider-than-code
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: landing-describer-lacks-command-failure-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: final-summary-spec-pointer-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings (F1 to F7), in the order of the keys above.
F1 is the only gating one: the command-failure-only note is not pinned against the flaky header.
F2 and F3 are LOW design findings about the fence length and the gate fixture.
F4 to F7 are consistency findings about a log key, the rule's wording, the describer stencil and a spec pointer.
