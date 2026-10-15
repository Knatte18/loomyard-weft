---
round: 3
ledger:
  - key: settler-rule-2-shadows-orch-settles
    rounds: [1]
    status: resolved
  - key: settle-record-reading-outside-allowlist
    rounds: [1]
    status: resolved
  - key: sleep-subtraction-vs-phase-sum
    rounds: [1, 2]
    status: resolved
  - key: formatseconds-drops-seconds-from-one-day
    rounds: [1]
    status: resolved
  - key: partial-runs-and-prewindow-buckets-open
    rounds: [1]
    status: resolved
  - key: driver-resume-gets-no-label
    rounds: [1]
    status: resolved
  - key: rejected-alternative-in-settler-rationale
    rounds: [1]
    status: resolved
  - key: settle-record-to-stop-join-unstated
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: non-go-rule-settle-records-unclassified
    rounds: [2]
    status: resolved
  - key: batten-file-record-matches-no-stop
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: interval-from-previous-stamp-recovers-time
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: notes-tokencount-interval-rule-claim
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: settle-records-for-run-in-flight
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: item-6-shed-label-unmapped
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven keys and round 2 three; round 3 raised six.
Sleep and the unclassified non-Go settle records were not raised again and are resolved.
The settle-record to stop join came back: its rule is stated now, but F1 says it cannot be computed for a lagging batten observation, a note-only stop or an empty producer.
Five keys are new in round 3: batten-file records with no matching stop, the interval origin after an out-of-order stamp, the Notes' tokencount claim, records for a run in flight, and the `shed` label in item 6.
