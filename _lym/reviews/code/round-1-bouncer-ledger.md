---
round: 1
ledger:
  - key: goto-hook-row-wanterr-too-loose
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1, mapped to `goto-hook-row-wanterr-too-loose`.
The new goto-hook row asserts only the shared stop-failure prefix, which a runner that was never built also yields.
The review gives the exact substring and a rename, so the fix is a local correction.
