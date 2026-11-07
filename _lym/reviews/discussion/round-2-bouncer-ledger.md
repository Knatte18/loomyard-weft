---
round: 2
ledger:
  - key: build-detached-push-unbounded
    rounds: [1]
    status: resolved
  - key: refusing-tb-bound-overclaims
    rounds: [1]
    status: resolved
  - key: reftable-fallback-list-only
    rounds: [1]
    status: resolved
  - key: git-floor-worktree-behaviour-deferred
    rounds: [1]
    status: resolved
  - key: noticesite-move-has-no-edited-file
    rounds: [1]
    status: resolved
  - key: d4-five-keys-bare-initialisers
    rounds: [1]
    status: resolved
  - key: d7-board-ahead-of-upstream-exemption
    rounds: [1]
    status: resolved
  - key: d2-upstream-check-misses-bare-commit
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-reader-directory-discovery-unspecified
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-build-inherits-process-global-seams
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-clean-board-claim-beamhub
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-tier-binaries-only-landed
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings; none reappears in round 2, so all seven are resolved.
Round 2 raised five new findings.
Three are design-class MEDIUM: the D2 upstream check misses a commit made straight into the records bare (F1), D1 leaves open how the reader finds the repository from a caller's directory (F2), and a build inherits its first caller's process-global seams (F3).
Two are LOW consistency findings with local fixes: D3's clean-board claim fails for loomcli's `beamHub` (F4), and `gate-tier-binaries-only` has landed on main (F5).
