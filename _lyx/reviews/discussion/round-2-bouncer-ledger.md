---
round: 2
ledger:
  - key: failed-write-ownership-unbounded
    rounds: [1]
    status: resolved
  - key: clear-removes-pause-request-unbounded
    rounds: [1]
    status: resolved
  - key: pid-liveness-and-spawn-exclusivity
    rounds: [1]
    status: resolved
  - key: unwatched-envelope-staleness
    rounds: [1]
    status: resolved
  - key: loop-key-set-disagreement
    rounds: [1]
    status: resolved
  - key: teardown-kills-loop-after-reed-session
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: failure-stop-invariant-domain-too-wide
    rounds: [1]
    status: resolved
  - key: condition-lifetime-never-fires
    rounds: [1]
    status: resolved
  - key: second-failure-rule-ambiguous
    rounds: [1]
    status: resolved
  - key: pattern-cap-entry-misfiled
    rounds: [1]
    status: resolved
  - key: d10-restates-d4-stderr
    rounds: [1]
    status: resolved
  - key: loop-dies-without-envelope
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: condition-overwrite-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: condition-ownership-class-mislabel
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: interrupted-error-omits-run-id
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: empty-key-drop-scope-disagreement
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: envelope-path-no-full-record
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eleven findings; round 2 did not re-raise ten of them, and its focus verdicts record the write-ownership, liveness, envelope-currency and key-set questions as bounded or settled.
The teardown key stays open: round 2 found the kill still leaves the in-flight child running past the session end.
Round 2 added a loop that dies without an envelope (BLOCKING), an unbounded overwrite of recorded stop conditions (MEDIUM), and four LOW consistency items on ownership class, `run_id`, empty-key scope and `envelope_path`.
