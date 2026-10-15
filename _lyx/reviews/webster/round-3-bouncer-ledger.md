---
round: 3
ledger:
  - key: follow-set-option-bare-exact-target
    rounds: [2]
    status: resolved
  - key: resume-viewer-not-on-reopened-own-window
    rounds: [2]
    status: resolved
---

Round 1 raised no findings.
Round 2 raised two BLOCKING design findings from the Publish failure: follow's `set-option` used a bare `=name` target tmux refuses, and the Resume test expected the viewer on the wrong window after a server restart.
Round 3 raised no findings: the review confirms the session-option targets now use `=name:` and the Resume test expects the viewer on the newest relaunched own window.
Both keys are resolved.
