---
round: 3
ledger:
  - key: empty-start-command-no-outcome
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-listing-change-unstated
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-omits-negative-bounds
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pause-expires-held-request
    rounds: [1, 2]
    status: resolved
  - key: shared-window-pane-moved-in-session
    rounds: [1, 2]
    status: resolved
  - key: marker-removals-escape-lock
    rounds: [1, 2]
    status: resolved
  - key: withdraw-refusal-keyed-on-phase
    rounds: [1, 2]
    status: resolved
  - key: reconcile-erases-pane-id-d2-reads
    rounds: [2]
    status: resolved
  - key: grace-spawns-ps-per-read
    rounds: [2]
    status: resolved
  - key: d7-not-git-repo-skip-unbounded
    rounds: [2]
    status: resolved
  - key: pause-notice-without-live-watcher
    rounds: [2]
    status: resolved
  - key: d811-holder-state-unverified
    rounds: [2]
    status: resolved
  - key: tmux-fact-duplicated-with-review-narrative
    rounds: [2]
    status: resolved
  - key: d3-double-spawn-bound-misnamed
    rounds: [2]
    status: resolved
  - key: removal-edges-unbounded-condition3
    rounds: [1]
    status: resolved
  - key: d2-guard-already-exists
    rounds: [1]
    status: resolved
  - key: missing-launch-time-premise
    rounds: [1]
    status: resolved
  - key: tmux-empty-answer-unverified
    rounds: [1]
    status: resolved
  - key: unnamed-false-live-cases
    rounds: [1]
    status: resolved
  - key: landing-conflict-stop-skips-shell-only
    rounds: [1]
    status: resolved
  - key: d6-no-reclaim-inside-grace
    rounds: [1]
    status: resolved
  - key: d7-skip-misses-post-resolve-teardown
    rounds: [1]
    status: resolved
  - key: acceptance-omits-windows-skip
    rounds: [1]
    status: resolved
  - key: child-shell-risk-lacks-evidence
    rounds: [1]
    status: resolved
  - key: departure-restates-rejected-alternative
    rounds: [1]
    status: resolved
---

Round 3 raised three new keys and re-raised none of the recurring ones, so the four keys open in rounds 1 and 2 are resolved, along with the rest of round 2's keys.
F1 (`empty-start-command-no-outcome`) is a gap in D1 condition 3: a pane started without the pinned default-command has an empty `pane_start_command` and the record names no outcome.
F2 and F3 are local consistency corrections: D1's listing change, and acceptance criteria for the negative bounds of D6, D7 and D3.
