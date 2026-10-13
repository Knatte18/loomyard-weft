---
round: 1
ledger:
  - key: d3-twill-detection-unpersisted-record
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-in-flight-rule-split-assistant-messages
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: real-run-audit-in-auto-mode-assumptions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.

- F1 (`d3-twill-detection-unpersisted-record`): D3 takes twill-ness from a spawn record that holds no fork authorization, and the attach branch rebuilds the spec from a fresh `plan_producer` read, so the "config edit changes nothing" claim is false.
  The fix names a persisted run-record field, or drops the claim.
- F2 (`d2-in-flight-rule-split-assistant-messages`): D2 classifies by last content block, but split assistant messages share the final `stop_reason` across lines, leaving some lines neither in flight nor over.
  The fix classifies by stop reason and adds explicit criteria rows.
- F3 (`real-run-audit-in-auto-mode-assumptions`): an audit result sits in Auto-mode assumptions and duplicates `support-log.md` and Scope Out; delete the bullet.
