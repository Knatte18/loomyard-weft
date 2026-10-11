---
round: 2
ledger:
  - key: d5-exit-unattached-kills-server-before-pin
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-destroy-unattached-kills-session-before-pin
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-server-starting-boot-undefined
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-glob-metacharacters-in-path
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: foreign-sessions-keep-server-alive
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: d4-parse-error-applies-nothing
    rounds: [1]
    status: resolved
  - key: d5-open-ended-pin-clause
    rounds: [1]
    status: resolved
  - key: tmux-conf-absence-bullet-misfiled
    rounds: [1]
    status: resolved
---

Round 1's blocking pair was the undefined server-starting boot and the `destroy-unattached on` ordering failure.
Round 2 shows both only partly fixed: the candidate rule still misses a server that dies between check and spawn, and the pin ordering has a window between spawn and pin.
Probing the pin ordering turned up a worse defect: `exit-unattached on` ends the server before a separate post-claim pin can run, which makes the third acceptance criterion unpassable.
The glob rule and the foreign-session rule narrowed to LOW: the claim's nested command string needs a path rule, and D8 needs a rule over every session-listing surface.
D4's parse-error behavior, the pin clause and the misfiled bullet no longer apply.
