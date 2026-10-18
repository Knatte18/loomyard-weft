---
round: 2
ledger:
  - key: implementers-second-registry-map
    rounds: [1]
    status: resolved
  - key: implementer-config-passthrough-untested
    rounds: [1]
    status: resolved
  - key: implementer-template-default-untested
    rounds: [1]
    status: resolved
  - key: implementer-setting-nonstring-refusal
    rounds: [1]
    status: resolved
  - key: creel-test-change-unlicensed
    rounds: [1]
    status: resolved
  - key: summary-claims-reverted-creel-fix
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pattern-unrelated-invariants-reworded
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1's five findings no longer appear in round 2; the review checked the registry map, the pass-through, the template default and the setting refusals and found them sound.
Round 2 raised two non-gating findings: the change description names a creel fix the final diff does not contain, and `PATTERN.md` rewords three unrelated invariants.
