---
round: 3
ledger:
  - key: d6-reprompt-retry-never-fires
    rounds: [1]
    status: resolved
  - key: stopped-outside-file-contract-first
    rounds: [1]
    status: resolved
  - key: d1-unmatched-turn-start-holds-idle-pane
    rounds: [1]
    status: resolved
  - key: d4-no-work-done-premise-unbounded
    rounds: [1]
    status: resolved
  - key: d2-settle-wait-vs-d3-window
    rounds: [1]
    status: resolved
  - key: stopped-overwrite-vs-burler-criterion
    rounds: [1, 2]
    status: resolved
  - key: stop-verb-record-write-failure
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-gaps
    rounds: [1]
    status: resolved
  - key: q8-open-while-d4-decided
    rounds: [1]
    status: resolved
  - key: restated-content-in-decision-record
    rounds: [1]
    status: resolved
  - key: failed-send-leaves-text-blocks-retry
    rounds: [2]
    status: resolved
  - key: d6-unmatched-turn-start-no-override
    rounds: [2]
    status: resolved
  - key: d6-overshoot-bound-undefined-appearance-check
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: completion-signal-tripwire-misses-stopped-write
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: constraint-contradicts-scope-on-attach
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: d1-turn-start-interrupt-capability-misplaced
    rounds: [2]
    status: resolved
  - key: d8-text-not-scoped-to-outside-shuttle
    rounds: [2]
    status: resolved
  - key: d7-mutex-claim-vs-record-write
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: text-never-appeared-error-class
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
  - key: d3-clear-premise-engine-limit
    rounds: [3]
    status: open
    class: design
    severity: LOW
  - key: d6-restore-trigger-unstated
    rounds: [3]
    status: open
    class: decision
    severity: LOW
  - key: d1-criterion-drops-every-poll
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 and 2 left a set of findings that round 3 no longer reproduces: the retry that never fires, the missing override for an unmatched `turn_start`, the text left in the box after a failed send, and the burler criterion.

Round 3 raised eight findings.
The BLOCKING `design` one is F1: D7's claim that the stop mark is the only field shared with `Wait` is false, because the stop verb's record write shares `run.state` and `run.json` with `Wait`'s own saves.
Other open items: the last confirm read breaks D3's loop and overshoot bound (F2), the Constraints audit one `stopped` site where D7 writes two (F3), the error class of a text that never appeared (F4), the clear's premise holding only for some engines (F5), the restore trigger in D6 (F6), the attach Constraint (F7) and D1's criterion wording (F8).
