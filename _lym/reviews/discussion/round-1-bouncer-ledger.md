---
round: 1
ledger:
  - key: placeholders-are-worktree-roots-not-anchors
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: env-removal-false-premise-unbounded-direct-call
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: nonzero-exit-opens-view-mode
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: mistyped-buttons-group-fails-config-loads
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-placeholder-absent-or-landing-parent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cancel-task-session-cwd-blocks-removal-windows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: percent-strftime-expanded-in-label-and-confirm
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-d4-disagree-on-failed-desktop-load
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lazy-beam-vs-closure-contract
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bound-names-too-few-button-config-files
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-bound-covers-only-booted-sessions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: opus-high-unquoted-glob
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 is the first review; the previous ledger is `(none)`.
Twelve findings are open.
The gating ones are two BLOCKING design findings, namely worktree-root placeholders that `lymcwd.Resolve` refuses in a subpath-anchored hub, and an env-removal premise that is false (the tmux server already strips the strand variables), which leaves a direct `lym reed button --confirmed` call unbounded.
Four MEDIUM design findings follow: exit code and view mode, a mistyped `buttons` group failing typed config loads, an unstated `{beam}` for an absent or landing parent, and a Windows cwd lock on CANCEL from a task session.
The rest are LOW consistency or small design corrections and one NIT.
