---
round: 3
ledger:
  - key: file-return-leaves-stale-store-and-parts
    rounds: [1]
    status: resolved
  - key: codex-cache-write-risk-settled-by-repo
    rounds: [1]
    status: resolved
  - key: second-price-table-beside-engine-pricer
    rounds: [1]
    status: resolved
  - key: part-records-no-path-into-board-store
    rounds: [1]
    status: resolved
  - key: auto-approve-key-read-time-unbounded
    rounds: [1]
    status: resolved
  - key: approval-survives-return-to-raddle-write
    rounds: [1]
    status: resolved
  - key: no-cold-proposal-in-auto-mode
    rounds: [1]
    status: resolved
  - key: no-waiting-notice-after-return
    rounds: [1, 2]
    status: resolved
  - key: entry-done-mid-run-hides-later-halts
    rounds: [1, 2]
    status: resolved
  - key: sketch-promote-bound-breaks-on-label-drop
    rounds: [1]
    status: resolved
  - key: gone-from-board-read-as-landed
    rounds: [1]
    status: resolved
  - key: pattern-changes-no-coverage-check
    rounds: [1]
    status: resolved
  - key: cost-fit-ignores-started-discussed
    rounds: [1]
    status: resolved
  - key: runs-without-size-record
    rounds: [1]
    status: resolved
  - key: recheck-misses-relabelled-entries
    rounds: [1, 2]
    status: resolved
  - key: no-done-marker-for-step-3
    rounds: [1, 2]
    status: resolved
  - key: uncited-facts-and-notify-on-land-exception
    rounds: [1]
    status: resolved
  - key: raddle-store-route-to-origin
    rounds: [1, 2]
    status: resolved
  - key: usage-record-pattern-text-overreaches
    rounds: [1]
    status: resolved
  - key: stencil-requirement-filed-as-open-risk
    rounds: [1]
    status: resolved
  - key: round-usage-second-token-writer
    rounds: [2]
    status: resolved
  - key: beam-parts-released-before-beam-exists
    rounds: [2]
    status: resolved
  - key: per-message-pricing-has-no-source
    rounds: [2]
    status: resolved
  - key: carried-criterion-names-unbuilt-decisions
    rounds: [2]
    status: resolved
  - key: slug-freshness-ignores-raddle-store
    rounds: [2]
    status: resolved
  - key: codex-output-includes-reasoning-unsettled
    rounds: [2]
    status: resolved
  - key: store-slug-reuse-by-non-raddle-creators
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: step-5-marker-lost-when-released-part-lands
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risk-defers-beam-up-reinvoke-policy
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: usage-writer-has-no-failure-rule
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 3 mapping: F1 to `store-slug-reuse-by-non-raddle-creators`, F2 to `step-5-marker-lost-when-released-part-lands`, F3 to `open-risk-defers-beam-up-reinvoke-policy`, F4 to `usage-writer-has-no-failure-rule`.

Every round 2 key not met by a round 3 finding is resolved.
The review's Rejected section shows the Raddle-File resume after step 3, the marker against prune, the release order, the store's staging and replay, the usage pattern fit, the pricing source and the D6 and D8 checks as held.
`slug-freshness-ignores-raddle-store` is resolved: D2, D5, D6 and D8 now check a raddle filing's slugs against the store, and F1 is a different break, the board's own new-slug rule ignoring the store, tracked under its own key.
`no-done-marker-for-step-3` is resolved: step 3's marker holds, and F2 is the same failure at step 5, tracked under its own key.

The loop has shrunk from twenty findings to eleven to four, with no BLOCKING finding left.
The two open MEDIUM findings are first raised this round, so Go carries them into `## Open risks`.
