---
round: 1
entries:
  - key: origin-wait-first-pass-unbounded-respawn
    status: closed
    finding: F1
    reason: "64ac643b5 and d660183b0: oweCatchUpForStart (starts.go) makes an origin-wait first pass a refused start with backoff when there is no catch-up or one succeeded since the entry's last such stop (kept in daemon memory, not StartState); both test rows added and the change description's bullet corrected"
---
