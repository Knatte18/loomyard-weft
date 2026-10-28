---
round: 1
ledger:
  - key: oversized-first-message-overflows-hook-field
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: continuation-count-climbs-reset-unordered
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: daemon-restart-loses-count-and-woken-marks
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: idle-reading-missing-from-delivery-seam
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bell-misdirects-pre-deploy-sessions
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: merged-stop-command-acks-before-claude-reads
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: blocked-stop-stamp-line-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: post-interrupt-hand-added-strand
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-rationale-false-for-hand-added-strands
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d9-sentence-shares-deny-notice-flag
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: windows-quote-delivery-unresearched
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: non-claude-sessions-lose-creel-posts
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, all still open.
The blocking one is that a message over the hook field cap is delivered as a preview while creel journals it read.
The MEDIUM design findings concern the continuation count and its reset order, daemon-restart state, the missing idle reading in the Delivery seam, and the bell reaching pre-deploy sessions.
The LOW findings are local gaps in D2, D6, D7 and D9 and one unresearched Windows quoting fact; F12 is a question for the parent about non-Claude sessions.
