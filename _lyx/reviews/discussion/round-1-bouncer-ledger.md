---
round: 1
ledger:
  - key: acceptance-criteria-contradict-stall-notes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stored-but-unnamed-wider-than-timeout
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-stderr-line-routing-unspecified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-mislabels-state-changing-verbs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cobra-refusals-bypass-fallback
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five LOW findings, all with text-level fixes the findings spell out.
Acceptance criteria 3 and 4 ignore the stall notes that stay on stderr.
The stored-but-unnamed gap covers every error-built refusal after the store, not only the wait timeout.
D1 needs a sentence that its line bypasses `writeRefusal`, and its "stored nothing" label is wrong for interrupt, forward, hold and release.
Cobra-level refusals keep their current behaviour and D5's doc line must say so.
