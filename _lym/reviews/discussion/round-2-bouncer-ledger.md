---
round: 2
ledger:
  - key: verb-tests-write-real-machine-file
    rounds: [1]
    status: resolved
  - key: collision-check-shared-identity
    rounds: [1]
    status: resolved
  - key: repair-to-own-name-refused
    rounds: [1]
    status: resolved
  - key: hub-pause-advice-unfounded
    rounds: [1]
    status: resolved
  - key: owner-pause-ends-processes
    rounds: [1]
    status: resolved
  - key: sweep-misses-origin-only-owners
    rounds: [1]
    status: resolved
  - key: no-owner-for-file-path-and-reader
    rounds: [1]
    status: resolved
  - key: relative-xdg-config-home
    rounds: [1]
    status: resolved
  - key: slack-rename-back-unsafe
    rounds: [1]
    status: resolved
  - key: done-beams-keep-old-owner
    rounds: [1]
    status: resolved
  - key: lost-former-names-drop-landings
    rounds: [1]
    status: resolved
  - key: lift-before-sweep-skips-pair-pushes
    rounds: [1]
    status: resolved
  - key: former-names-unvalidated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hostname-name-not-in-names
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: error-wrappers-contradict-way-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: no-fallback-overstates-error-stop
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: collision-blind-to-ownerless-machine
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lost-entries-no-report-line
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: committed-unpushed-sweep-flaps-dobby
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: goal-no-change-fails-relative-home
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: d1-directory-claim-vs-d2
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: auto-mode-assumptions-omit-d4-d8
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-background-edits-in-notes
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d8-windows-test-env-claim
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised twelve findings, led by the BLOCKING one that D8 isolated only `internal/machinename`'s tests.
Round 2 re-raised none of them, so all twelve are marked resolved.

Round 2 raised twelve new findings with no BLOCKING among them.
The three MEDIUM design findings concern unvalidated former names, the hostname rule's name missing from `names`, and caller wrappers appending a contradicting hostname fix.
They are first raised in round 2, so Go carries them into Open risks.
The rest are LOW bounds and wording corrections, plus one NIT on D8's Windows claim.
