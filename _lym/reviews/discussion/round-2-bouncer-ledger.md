---
round: 2
ledger:
  - key: d6-fabric-ide-knowledge
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-out-carries-history
    rounds: [1]
    status: resolved
  - key: d7-readme-folds-survive-parent-drop
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: open-risks-settled-by-parent
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-board-path-bypass-misstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-anywhere-in-worktree-overclaim
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 2 resolved most of round 1's findings.
The two parent questions, the phone goal and the ide-prime-setup remainder, were answered by added entries and are resolved here.
Their leftovers in `## Open risks` and in D7 come back as new consistency keys.

Still gating: the fabric boundary, now over the workspace splice rather than tasks (`d6-fabric-ide-knowledge`, BLOCKING),
and the unproven failure visibility of the `board open` task (`board-open-refusal-invisible`, MEDIUM).

Round 1 keys not raised again, including the link allowlist, subdirectory resolution, the beam start binary and the mixed-version bound, are resolved.
`d1-names-no-escape` recurs as a LOW: the fix did not reach the file.
