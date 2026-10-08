---
round: 2
ledger:
  - key: dotfill-cross-client-test-geometry
    rounds: [1]
    status: resolved
  - key: driverbootstrap-stub-color-line
    rounds: [1]
    status: resolved
  - key: stale-statusline-comments-lifecycle
    rounds: [1]
    status: resolved
  - key: burlercli-wiring-test-reponame-header
    rounds: [1]
    status: resolved
  - key: reedengine-doc-strand-options-reassert
    rounds: [1]
    status: resolved
  - key: crosscontrol-sizing-comment-mismeasured
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
  - key: setwaitmark-doc-names-status-line
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised two BLOCKING `scope` findings (the two failing `tmux`-tier tests) and three comment findings.
Round 2's fresh review found both tests green on HEAD and did not re-raise any round-1 key, so all five are resolved.

Round 2 raised two NIT `consistency` findings, neither gating.
F1 is the `CrossClientControl` comment, whose claim about what the 12-row pane shows contradicts a measured capture.
F2 is the `SetWaitMark` doc, which still says "status line" where `doc.go` says the status bar renders the mark on the strand's button.
