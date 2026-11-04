---
round: 1
ledger:
  - key: sweep-returns-unread-mail-for-unlisted-hub-orch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: done-entry-notice-ignores-owner-machine
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-end-to-end-demonstration-of-landing-notice
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: failed-clear-mode-read-restart-rows-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: nil-clear-seam-repeats-line-every-tick
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-parent-aborts-whole-task-read
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prune-help-and-prose-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: landed-notice-fires-on-unrelated-edits
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: creel-row-covers-one-orch-name-form
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: redundant-failed-post-retry-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hand-marked-done-entry-noticed-as-landed
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised F1 to F11, each mapped to the key of the same position above.
F1 (BLOCKING) and F2 (MEDIUM) are the gating `design` findings: F1 changes what the removed-holder sweep does with a hub orch's unread mail, and F2 changes which machine's dobby may act on a done flagged entry, so both need a fresh review of the fix.
F3 and F4 are MEDIUM `scope` findings asking for tests and a change-description correction.
F5, F6 and F8 are LOW `design` corrections whose text fixes the result; F7, F9 and F10 are LOW; F11 is a NIT.
No key recurs, since this is the first round.
