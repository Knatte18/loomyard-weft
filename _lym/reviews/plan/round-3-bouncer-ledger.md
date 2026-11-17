---
round: 3
ledger:
  - key: card-23-dead-ownerpause-vars
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: dobby-hand-over-trigger-reowes-catch-up
    rounds: [1, 2]
    status: resolved
  - key: card-15-hold-refusal-not-before-pause-clear
    rounds: [2]
    status: resolved
  - key: card-23-bring-up-seam-unspecified
    rounds: [2]
    status: resolved
  - key: tender-resume-refuses-on-held-before-hold-rules
    rounds: [1]
    status: resolved
  - key: cards-8-23-unlisted-tests-on-old-seams
    rounds: [1]
    status: resolved
  - key: card-16-held-refusal-shadowed-by-refuse-by-act
    rounds: [1]
    status: resolved
  - key: dobby-start-registration-refusal-no-notice-no-status
    rounds: [1]
    status: resolved
  - key: machine-names-of-reads-registered-name-as-own
    rounds: [1]
    status: resolved
  - key: parent-lock-exemption-from-bolt-hold-uncarried
    rounds: [1]
    status: resolved
  - key: card-19-commit-run-intent-resign-arrow
    rounds: [1]
    status: resolved
  - key: card-17-forced-pair-removal-no-bound
    rounds: [1]
    status: resolved
  - key: owner-pause-text-stays-after-card-23
    rounds: [1]
    status: resolved
  - key: card-11-pattern-cli-cobra-machinecli-line
    rounds: [1]
    status: resolved
  - key: absences-decision-false-premise
    rounds: [1]
    status: resolved
---

Round 1 raised twelve findings, three BLOCKING; round 2 cut that to three MEDIUM findings.
Round 3 raised one LOW scope finding: card 23 leaves the unexported `ownerPause` and `ownerPauseMu` vars in `internal/orchcli/ownerpause.go` as dead code.

The round-3 review settled the three round-2 keys:
the `hand_over` trigger no longer re-owes the catch-up (the `handedOver` map),
card 15's refusal comes before `req.Clear`,
and card 23 names its replacement seams.
Only the new dead-vars key stays open, and its fix is mechanical: two Delete lines on card 23.
