---
round: 2
ledger:
  - key: watchdog-exemption-left-open
    rounds: [1]
    status: resolved
  - key: done-stamp-ignores-goto
    rounds: [1]
    status: resolved
  - key: stop-vs-checkout-order
    rounds: [1]
    status: resolved
  - key: flag-unreadable-before-session-start
    rounds: [1]
    status: resolved
  - key: d4-describe-bound-false
    rounds: [1]
    status: resolved
  - key: acceptance-three-skipped-rows
    rounds: [1]
    status: resolved
  - key: d6-reroute-never-reenters
    rounds: [1]
    status: resolved
  - key: d3-codereviewoff-no-reader
    rounds: [1]
    status: resolved
  - key: publish-guard-steer-text-false
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d7-bound-miscites-run-shed
    rounds: [1]
    status: resolved
  - key: entry-path-live-session-not-stopped
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-admitted-strand-reopen
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-bound-own-diff-misquote
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings.
Round 2 did not re-raise nine of them, and its focus questions on the Done stamp and the watchdog list held, so those are resolved.
The wake path's stop-versus-checkout order is resolved too; round 2 found the same hazard only on D6's entry path, filed as a new key.

`publish-guard-steer-text-false` is still open and now MEDIUM: D5 promises a session-specific steer and blocked forms that the guard table does not provide.

New in round 2: the entry path ending Done without stopping a live session (MEDIUM design), D4's "always the live one" claim against a session reopened outside the row (LOW consistency), and D4's Bound misquoting the caller rule's package doc (NIT consistency).
