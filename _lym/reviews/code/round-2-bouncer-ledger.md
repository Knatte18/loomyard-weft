---
round: 2
ledger:
  - key: repeated-name-literals-named-constants
    rounds: [1]
    status: resolved
  - key: failed-compose-errors-round
    rounds: [1]
    status: resolved
  - key: zero-assist-timeout-escapes-cap
    rounds: [1]
    status: resolved
  - key: wait-error-ending-no-test-row
    rounds: [1]
    status: resolved
  - key: await-cap-resolution-untested
    rounds: [1]
    status: resolved
  - key: per-helper-test-duplicates-rows
    rounds: [1]
    status: resolved
  - key: overview-omits-assist-config-keys
    rounds: [1]
    status: resolved
  - key: keep-warm-cap-wrong-chair-role
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: timeout-knob-message-wrong-zero-meaning
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings; round 2 raises none of them again, so all seven are resolved.
Round 2 found that the assistant's keep-warm cap is resolved for the wrong chair role, a BLOCKING design finding whose fix reworks the wiring from one cap to one per review segment.
It also found a LOW wording error in the shared timeout-knob refusal, whose fix is local.
