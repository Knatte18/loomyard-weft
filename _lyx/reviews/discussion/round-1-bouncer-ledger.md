---
round: 1
ledger:
  - key: d8-precompare-wrong-bytes
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: d8-race-bound-second-window
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-batten-step-seed-site
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: d4-bound-vanished-batten-status
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-malformed-value-unspecified
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d5-byte-identity-false-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-retire-legacy-envelope-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d2-nonexistent-self-backlink
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-rationale-restates-a1
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings against `decision-record.md`.
D8 has two: the pre-rename compare names the wrong bytes (BLOCKING), and the race bound omits the window after the post-rename re-read.
D4 has two: a second fresh-status seed site in batten, and a bound that fails when the batten status vanishes.
D5 has two: the malformed-value case is open, and the byte-identity premise is false.
D6's dry-run envelope does not match `retire-legacy`'s applied envelope.
D2 has a nonexistent self back-link (LOW) and a Rationale sentence that restates A1 (NIT).
No earlier ledger exists, so every key is new.
