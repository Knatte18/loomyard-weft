---
round: 3
ledger:
  - key: card5-pushclassified-records-takeover
    rounds: [1]
    status: resolved
  - key: deferred-records-side-reads-as-landed
    rounds: [1]
    status: resolved
  - key: branch-rewind-outside-destroy-chokepoint
    rounds: [1, 2]
    status: resolved
  - key: card8-spawn-field-tests-not-targets
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: leased-code-branch-moves-executor-unnamed
    rounds: [2]
    status: resolved
  - key: readopt-commitfailed-calls-removed-spawn
    rounds: [2]
    status: resolved
  - key: failed-push-slot-loses-second-runs-owed-push
    rounds: [2]
    status: resolved
  - key: dropped-intent-read-uses-stale-prime-ref
    rounds: [2]
    status: resolved
  - key: code-tip-read-from-every-warp-sha-commit
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: owed-tip-pushed-hook-unreachable-from-external-test
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: loomcommitstatusdeps-doc-describes-removed-spawn
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 16 findings, round 2 raised 7, round 3 raised 3.
Round 3 did not re-raise any earlier key, so all earlier keys are marked resolved.
Round 3 opened three new keys.
F1 (`code-tip-read-from-every-warp-sha-commit`) says the readers of a run's last-transition code tip must select transition commits by their own trailer, since every fabric weft commit carries `Warp-SHA`.
F2 (`owed-tip-pushed-hook-unreachable-from-external-test`) says card 6 needs a `Set...HookForTest` helper for `owedTipPushedHook`.
F3 (`loomcommitstatusdeps-doc-describes-removed-spawn`) says `loomCommitStatusDeps` keeps a doc comment for the removed `Spawn` field and no card targets it.
