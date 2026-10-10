---
round: 1
ledger:
  - key: failed-write-ownership-unbounded
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: clear-removes-pause-request-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: pid-liveness-and-spawn-exclusivity
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: unwatched-envelope-staleness
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: loop-key-set-disagreement
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: teardown-kills-loop-after-reed-session
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: failure-stop-invariant-domain-too-wide
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: condition-lifetime-never-fires
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: second-failure-rule-ambiguous
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: pattern-cap-entry-misfiled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d10-restates-d4-stderr
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, none carried from an earlier round.
The gating cluster is ownership and bounds in the `--until-stop` loop design: who may write `failed` to the status file, who may clear the pause request, how loop liveness and envelope currency are proved, and when teardown kills the loop.
The rest are key-set and invariant-scope consistency, two undecided edge cases, and two wording or placement items.
