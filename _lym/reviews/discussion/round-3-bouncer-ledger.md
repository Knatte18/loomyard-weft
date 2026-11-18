---
round: 3
ledger:
  - key: d7-relaunch-no-mechanism
    rounds: [1]
    status: resolved
  - key: pinned-no-entry-blocked-pin
    rounds: [1]
    status: resolved
  - key: hub-pin-switched-on-limit
    rounds: [1]
    status: resolved
  - key: d6-copied-transcript-resume
    rounds: [1]
    status: resolved
  - key: d7-vs-retryapierror
    rounds: [1]
    status: resolved
  - key: orch-watcher-cannot-restart
    rounds: [1, 2]
    status: resolved
  - key: d2-prune-no-input
    rounds: [1]
    status: resolved
  - key: blocked-until-vs-windows-eligibility
    rounds: [1]
    status: resolved
  - key: d5-forged-input-bound
    rounds: [1, 2]
    status: resolved
  - key: openlaunch-no-status-line
    rounds: [1]
    status: resolved
  - key: d6-openlaunch-copy-sentence
    rounds: [1]
    status: resolved
  - key: d3-reset-zero-weight
    rounds: [1]
    status: resolved
  - key: d7-switch-partial-counter
    rounds: [1]
    status: resolved
  - key: retired-account-no-move
    rounds: [1]
    status: resolved
  - key: config-dir-links-unresolved
    rounds: [1]
    status: resolved
  - key: d7-set-bound-reach
    rounds: [1]
    status: resolved
  - key: beams-pin-open-risk-writer-choice
    rounds: [1]
    status: resolved
  - key: rationales-reargue-rejected
    rounds: [1]
    status: resolved
  - key: d10-landing-notes-criterion
    rounds: [1]
    status: resolved
  - key: d7-relaunch-mark-unbounded
    rounds: [2]
    status: resolved
  - key: d7-relaunch-guid-orphans-callers
    rounds: [2]
    status: resolved
  - key: d7-relaunch-prompt-startup
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-copy-record-no-run-dir
    rounds: [2]
    status: resolved
  - key: d7-relaunch-record-after-reed
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-running-readers-skip-relaunching
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised nineteen findings and round 2 six; round 3 resolves five of round 2's keys and leaves one open.
The mark's bound, the unchanged strand GUID, the D6 copy record's run directory, the orch watcher's restart and D5's forged-input bound are settled.
`d7-relaunch-prompt-startup` is still open: the first-turn half is settled, but D7 step 4's startup-gate mechanism, `awaitStartup`, passes vacuously on the reused events file.
Two new BLOCKING design findings join it, both in D7's relaunch window:
reed relaunches the pane before the record is written, so a dying Wait leaves a live pane under a stale relaunching record,
and the shed stop chain plus other readers of a running record skip a relaunching run.
