---
round: 1
ledger:
  - key: relaunched-writer-forks-lack-files
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1: a writer relaunched after its process died before its Agent call may hold none of the reviewed files in context, yet its forks are told never to read them again.
The fix is a sentence in the relaunch paragraph of `contracts/stencils/twill/twill-directive.md` telling the writer to read back every reviewed file it did not write, so the finding text fixes the result.
