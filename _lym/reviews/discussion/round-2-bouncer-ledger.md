---
round: 2
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
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-prune-no-input
    rounds: [1]
    status: resolved
  - key: blocked-until-vs-windows-eligibility
    rounds: [1]
    status: resolved
  - key: d5-forged-input-bound
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-relaunch-guid-orphans-callers
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d7-relaunch-prompt-startup
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-copy-record-no-run-dir
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1 raised nineteen findings; round 2 resolved seventeen of them and carries two.
F6 (`orch-watcher-cannot-restart`) is carried as F24: the watcher's restart through `resumeOrch` leaves the old strand and orch state behind.
F9 (`d5-forged-input-bound`) is carried as F25: the forged-input bound omits D7's switch.
F1 is resolved, but the relaunch mechanism it asked for has new defects, filed as F20 to F22 under their own keys:
an unbounded relaunching mark (F20), a new strand GUID that orphans callers holding the old one (F21), and a resume launch that re-types the task or a skill load and skips startup (F22).
F23 is a separate D6 defect: the copy record has no run directory on the start path.
