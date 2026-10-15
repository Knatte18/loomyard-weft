---
round: 2
ledger:
  - key: follow-set-option-bare-exact-target
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: resume-viewer-not-on-reopened-own-window
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised no findings.
Round 2 raised two BLOCKING design findings from the Publish failure.
F1 (`follow-set-option-bare-exact-target`): `set-option -t =name` is refused by tmux, so follow is never written; the fix is a local target change to `=name:` in `FollowRun`, `followSessionVia` and the test helpers.
F2 (`resume-viewer-not-on-reopened-own-window`): after a server restart `Resume` leaves the viewer on `@2` rather than the reopened own window; the cause is not yet pinned down, so the fix may change behavior or the test expectation.
