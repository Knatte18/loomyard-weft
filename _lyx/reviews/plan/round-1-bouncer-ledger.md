---
round: 1
ledger:
  - key: end-signals-reused-pid
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: remove-gone-worktree-skips-owner-guard
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: gated-arrival-ignores-missing-outputs
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: marker-pass-kills-merging-step
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: paused-parent-lock-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unknown-holder-way-forward-circular
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: webster-doc-old-shell-rule
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: register-lock-file-unspecified
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stopped-refusal-live-pair-retry
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: owned-elsewhere-warn-no-next-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reed-down-help-not-updated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eleven findings, none BLOCKING, seven MEDIUM.
Four design-class MEDIUMs gate: pid-reuse safety in `End`/`EndProcess` (card 2) and three refusal-path flaws in card 10 (marker pass, paused parent-lock way forward, unknown-holder way forward).
The marker-pass fix changes which holders pass, so it is marked `design-wrong` and needs a fresh review; the gone-worktree removal fix in card 11 changes a decision's reach and is marked the same.
