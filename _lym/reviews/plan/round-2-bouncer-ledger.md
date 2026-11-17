---
round: 2
ledger:
  - key: dobby-hand-over-trigger-reowes-catch-up
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card-15-hold-refusal-not-before-pause-clear
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card-23-bring-up-seam-unspecified
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
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

Round 1 raised twelve findings, three BLOCKING.
Round 2 raised three MEDIUM findings and cleared every round-1 area but one in its Rejected section.

The hand_over trigger stays open: round 1 found a no-pair loop, and round 2 finds that a leftover unregistered pair directory
still lists as `hand_over` with `PairPresent` true after a successful catch-up, so it owes again each tick.
The tender resume key is resolved, since the held refusal now reaches card 15's rules;
a separate ordering gap inside `Resume` (the refusal comes after the pause-intent clear) is filed as a new key.
Card 23's replacement seam for the retired owner pause is open as a decision-class finding.
