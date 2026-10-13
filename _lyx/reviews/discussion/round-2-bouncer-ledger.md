---
round: 2
ledger:
  - key: d3-twill-detection-unpersisted-record
    rounds: [1]
    status: resolved
  - key: d2-in-flight-rule-split-assistant-messages
    rounds: [1]
    status: resolved
  - key: real-run-audit-in-auto-mode-assumptions
    rounds: [1]
    status: resolved
  - key: d2-null-stop-reason-unnamed-reading
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-fork-audit-reads-attaching-spec
    rounds: [2]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 raised three findings: an unpersisted twill record in D3, the split-assistant-message rule in D2, and an audit result duplicated in Auto-mode assumptions.
Round 2 did not raise them again, so all three are resolved.

Round 2 raised two LOW findings, both new keys.
- F1 (`d2-null-stop-reason-unnamed-reading`): a message cut off mid-stream carries a null stop reason, which D2 gives no reading and no Acceptance row pins.
  The fix text names the reading, so it is mechanical.
- F2 (`d3-fork-audit-reads-attaching-spec`): the fork audit at finalize still reads the attaching Call's spec while the gate reads the recorded authorization.
  The fix needs a decision on which one the audit reads.
