---
round: 2
ledger:
  - key: card5-pushclassified-records-takeover
    rounds: [1]
    status: resolved
  - key: deferred-records-side-reads-as-landed
    rounds: [1]
    status: resolved
  - key: branch-rewind-outside-destroy-chokepoint
    rounds: [1, 2]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: card8-spawn-field-tests-not-targets
    rounds: [1, 2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: teardown-wait-error-type-mismatch
    rounds: [1]
    status: resolved
  - key: card7-owner-guard-narrows-d2-unrecorded
    rounds: [1]
    status: resolved
  - key: card7-origin-reset-no-bound
    rounds: [1]
    status: resolved
  - key: card6-batten-record-read-via-child-reader
    rounds: [1]
    status: resolved
  - key: card9-end-only-cancel-premise-false
    rounds: [1]
    status: resolved
  - key: code-tip-way-forward-mismatch
    rounds: [1]
    status: resolved
  - key: card12-gives-up-locally-held-tip
    rounds: [1]
    status: resolved
  - key: detached-teardown-status-push-unnamed
    rounds: [1]
    status: resolved
  - key: marktaskdone-resign-arrow-missing
    rounds: [1]
    status: resolved
  - key: fabriccli-doc-keeps-retired-respawn
    rounds: [1]
    status: resolved
  - key: card8-pushlockbusy-owed-without-record
    rounds: [1]
    status: resolved
  - key: card5-machinenames-second-construction-site
    rounds: [1]
    status: resolved
  - key: records-commit-carries-no-warp-sha-trailer
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: leased-code-branch-moves-executor-unnamed
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: readopt-commitfailed-calls-removed-spawn
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: failed-push-slot-loses-second-runs-owed-push
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: dropped-intent-read-uses-stale-prime-ref
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised 16 findings.
Round 2 re-raised two of those keys and did not repeat the other fourteen, so those are marked resolved.
The re-raised pair: `branch-rewind-outside-destroy-chokepoint` (round 2 F2, the parent reset still has no executor symbol) and `card8-spawn-field-tests-not-targets` (round 2 F5, the loomcli test that still calls the removed `Spawn`).
Round 2 F4 (the readopt seam's `CommitFailed`) is the same removed-`Spawn` family but a distinct site, kept as its own key.
New this round: the missing `Warp-SHA` trailer on seam commits (F1), the unnamed leased-move executor (F3), the per-slot failed-push record (F6) and the stale prime tracking ref (F7).
