---
round: 3
ledger:
  - key: d6-fabric-ide-knowledge
    rounds: [1, 2]
    status: resolved
  - key: board-open-subdir-resolve
    rounds: [1]
    status: resolved
  - key: d4-link-allowlist-undefined
    rounds: [1]
    status: resolved
  - key: d6-beam-start-binary-deferred
    rounds: [1]
    status: resolved
  - key: d6-mixed-version-bound
    rounds: [1]
    status: resolved
  - key: board-open-refusal-invisible
    rounds: [1, 2]
    status: resolved
  - key: parent-phone-goal
    rounds: [1]
    status: resolved
  - key: parent-ide-prime-setup-remainder
    rounds: [1]
    status: resolved
  - key: d6-tracked-tasks-json-misattributed
    rounds: [1]
    status: resolved
  - key: d1-projection-no-section-order
    rounds: [1]
    status: resolved
  - key: d4-design-file-link-may-not-exist
    rounds: [1]
    status: resolved
  - key: d5-second-keypress-active-task
    rounds: [1]
    status: resolved
  - key: d3-downgrade-replay-misstated
    rounds: [1]
    status: resolved
  - key: d1-names-no-escape
    rounds: [1, 2]
    status: resolved
  - key: scope-out-carries-history
    rounds: [1]
    status: resolved
  - key: d7-readme-folds-survive-parent-drop
    rounds: [2]
    status: resolved
  - key: open-risks-settled-by-parent
    rounds: [2]
    status: resolved
  - key: d5-board-path-bypass-misstated
    rounds: [2]
    status: resolved
  - key: d5-anywhere-in-worktree-overclaim
    rounds: [2]
    status: resolved
  - key: d4-other-fields-unescaped
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 3 raised one finding: D4 bounds HTML escaping for entry bodies but leaves the other fields rendered from `window.BOARD` unstated.
Every key open after round 2 was not raised again and is resolved.
The new key is first raised in the closing round, so it is carried into Open risks rather than blocking.
