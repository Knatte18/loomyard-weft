---
round: 2
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
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: entry-done-mid-run-hides-later-halts
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-done-marker-for-step-3
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: uncited-facts-and-notify-on-land-exception
    rounds: [1]
    status: resolved
  - key: raddle-store-route-to-origin
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: usage-record-pattern-text-overreaches
    rounds: [1]
    status: resolved
  - key: stencil-requirement-filed-as-open-risk
    rounds: [1]
    status: resolved
  - key: round-usage-second-token-writer
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: beam-parts-released-before-beam-exists
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: per-message-pricing-has-no-source
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: carried-criterion-names-unbuilt-decisions
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: slug-freshness-ignores-raddle-store
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: codex-output-includes-reasoning-unsettled
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 2 mapping: F1 to `entry-done-mid-run-hides-later-halts`, F2 to `round-usage-second-token-writer`, F3 to `beam-parts-released-before-beam-exists`, F4 to `raddle-store-route-to-origin`, F5 to `recheck-misses-relabelled-entries`, F6 to `no-done-marker-for-step-3`, F7 to `per-message-pricing-has-no-source`, F8 to `carried-criterion-names-unbuilt-decisions`, F9 to `slug-freshness-ignores-raddle-store`, F10 to `codex-output-includes-reasoning-unsettled`, F11 to `no-waiting-notice-after-return`.

Round 1 keys not met by a round 2 finding are resolved: the review's focus verdicts show the return handling, the approval mode and the engine pricer as held, and no round 2 finding touches the rest.
`usage-record-pattern-text-overreaches` is resolved; F2 is a different break, a second writer, and is tracked under its own key.

The open core is the Raddle-File step ordering: the run's entry marked done at step 3 (F1), the unstable step 3 marker (F6), the unshown route of the store to origin (F4), and beam parts released early (F3).
The cost design's remaining gaps are the round usage writer (F2) and the per-message source (F7).
