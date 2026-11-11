---
round: 2
ledger:
  - key: d2-generation-premises-false
    rounds: [1]
    status: resolved
  - key: d3-clear-reuses-ask-key
    rounds: [1]
    status: resolved
  - key: d5-cap-formula-unstated
    rounds: [1]
    status: resolved
  - key: d3-no-parent-unreachable
    rounds: [1]
    status: resolved
  - key: d4-answer-names-no-ask
    rounds: [1]
    status: resolved
  - key: d4-findings-file-shape
    rounds: [1]
    status: resolved
  - key: d6-ask-sender-not-loom
    rounds: [1]
    status: resolved
  - key: d2-key-flip-on-restart
    rounds: [1]
    status: resolved
  - key: d3-post-record-order
    rounds: [1]
    status: resolved
  - key: d3-d4-parent-re-resolved
    rounds: [1]
    status: resolved
  - key: d6-bound-overstates-guard
    rounds: [1]
    status: resolved
  - key: d4-no-parent-refusal-criterion
    rounds: [1]
    status: resolved
  - key: d5-timeout-line-criterion
    rounds: [1]
    status: resolved
  - key: d5-rejected-alternative-in-record
    rounds: [1]
    status: resolved
  - key: d7-parent-findings-unchecked
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-close-missed-on-handon-path
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-d7-chair-cannot-learn-ask-state
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-refusal-way-forward-misdirects
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-not-sent-rule-lacks-later-round
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-seed-stamp-precision
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-await-verb-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d2-unread-generation-facts
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's fourteen findings are all resolved: round 2 repeats none of them.
Round 2 found no BLOCKING finding.
Three MEDIUM design findings are new: D7's parent findings have no check, the ask's close and copy are missed on the accepted-review hand-on path, and the chair cannot learn the ask's state or report path.
The rest are LOW or NIT: refusal way forwards, a missing later-round condition, the seed stamp's precision, the await verb's bound, and unread generation facts.
