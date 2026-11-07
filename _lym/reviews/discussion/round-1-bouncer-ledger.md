---
round: 1
ledger:
  - key: build-detached-push-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: refusing-tb-bound-overclaims
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reftable-fallback-list-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: git-floor-worktree-behaviour-deferred
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: noticesite-move-has-no-edited-file
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d4-five-keys-bare-initialisers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d7-board-ahead-of-upstream-exemption
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none carried from an earlier round.
Four are design-class MEDIUM and gate: the build's detached push (F1), the refusing TB's overclaimed bound (F2), the websterengine gap in the reftable fallback (F3), and the git 2.43.0 worktree behaviour left to a later test (F4).
Three are consistency findings (F5, F6, F7) with local fixes.
