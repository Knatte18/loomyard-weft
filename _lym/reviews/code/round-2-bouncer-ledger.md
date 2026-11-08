---
round: 2
ledger:
  - key: draft-overwrites-existing-description
    rounds: [1]
    status: resolved
  - key: describe-unreadable-landing-record-untested
    rounds: [1]
    status: resolved
  - key: describe-draft-flag-wiring-untested
    rounds: [1]
    status: resolved
---

Round 1 raised three findings: `--draft` overwriting an existing description, and two test-coverage gaps.
Round 2 re-checked each against the fixed diff and found no defect, so all three are resolved.
