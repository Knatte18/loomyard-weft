---
round: 3
ledger:
  - key: d8-escalation-bypasses-judgment-refusal
    rounds: [1, 2]
    status: resolved
  - key: d8-settled-generation-rule-has-no-exit
    rounds: [1]
    status: resolved
  - key: d6-lens-reads-exceed-six-call-cap
    rounds: [1]
    status: resolved
  - key: d11-merge-in-fixer-attempt-record-lost
    rounds: [1]
    status: resolved
  - key: d1-landing-approval-is-parents-call
    rounds: [1]
    status: resolved
  - key: d8-hard-cap-reescalation-strips-decision-cause
    rounds: [1]
    status: resolved
  - key: d8-fix-row-silence-switch-unstated
    rounds: [1]
    status: resolved
  - key: d10-writer-wait-not-woken-and-silent-stop
    rounds: [1]
    status: resolved
  - key: d5-other-fans-and-hub-lens-layers
    rounds: [1]
    status: resolved
  - key: d8-accept-carries-answered-findings
    rounds: [1]
    status: resolved
  - key: d6-landed-commit-routes-two-ways
    rounds: [1]
    status: resolved
  - key: d8-d7-bound-is-prompt-text-only
    rounds: [1]
    status: resolved
  - key: d8-accept-exemption-misses-ledger-blocking-check
    rounds: [2]
    status: resolved
  - key: d8-continue-bypasses-dropped-key-guard
    rounds: [2]
    status: resolved
  - key: d8-refused-accept-forgets-parent-answer
    rounds: [2]
    status: resolved
  - key: d8-next-fixer-carrier-does-not-exist
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-added-after-discussion-snapshot-lost-on-retire
    rounds: [2]
    status: resolved
  - key: d10-no-parent-stop-defers-gate-behaviour
    rounds: [2]
    status: resolved
  - key: d10-timeout-bound-names-nonexistent-respawn
    rounds: [2]
    status: resolved
  - key: d8-refused-accept-not-a-continue-on-disk
    rounds: [2]
    status: resolved
  - key: d8-hard-cap-refusal-has-no-exit
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Rounds 1 and 2 raised and settled the D8 escalation, lens-read, Merge-In record and D10 findings; round 3 did not re-raise any of them, and its settled section confirms the accept checks, the dropped-key check, the snapshot file and the D10 no-parent stop.
Two D8 findings remain.
The carrier for a refused judgment's BLOCKING findings has been raised twice: the round-2 fix asserted a `continue` hands them to a fixer, but the perch order runs the fixer before the judge and a CONVERGED verdict writes an empty focus, so no row carries them.
The new finding is that a refusal still holding at or past the hard cap re-escalates with only accept offered, so the parent has no terminal exit.
Both need a design change in D8 and a fresh review of that text.
