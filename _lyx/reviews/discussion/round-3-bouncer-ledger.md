---
round: 3
ledger:
  - key: d5-marker-path-stale-or-unreachable
    rounds: [1, 2]
    status: resolved
  - key: d6-resume-partial-or-mixed-round
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: d1-d6-d7-join-gate-ordering-single-gate
    rounds: [1, 2]
    status: resolved
  - key: d8-disputed-criterion-presumes-checker
    rounds: [1, 2]
    status: resolved
  - key: d9-resolution-at-config-load
    rounds: [1]
    status: resolved
  - key: decisions-without-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: fixer-deadline-vs-waiting-time
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-rationale
    rounds: [1]
    status: resolved
  - key: d1-no-shuttle-capability-vs-stop
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: marker-standalone-path-and-cleanup
    rounds: [2, 3]
    status: open
    class: consistency
    severity: NIT
  - key: fixer-may-edit-review-file
    rounds: [2, 3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-first-terminal-event-wording
    rounds: [2]
    status: resolved
  - key: fixer-done-before-marker
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-bound-overstates-prompt-only-release
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 and 2 keys not raised in round 3 are resolved: the gate-ordering key (BLOCKING in round 2), the marker-path key, the disputed-criterion key and the first-terminal-event wording key.
Round 3 maps its findings as follows:

- F1 is new: `fixer-done-before-marker`, the only gating finding.
- F2 is the producer-side half of `d1-no-shuttle-capability-vs-stop`: the resume branch lacks the strand remover and the shared review-ready step.
- F3 is `fixer-may-edit-review-file` reopened at the judge's re-read of the review file.
- F4 is the wording remainder of `d6-resume-partial-or-mixed-round`.
- F5 is new: `d5-bound-overstates-prompt-only-release`.
- F6 is `marker-standalone-path-and-cleanup` reduced to a NIT.
