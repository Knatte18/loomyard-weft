---
round: 1
ledger:
  - key: quoting-hint-fragment-regex-preempts-colon-hint
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: routed-field-comments-name-stuck-alone
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings.
F1 (`quotingHint`): the fragment regex matches valid quoted summaries, so the colon hint never shows; the review's fix text pins the regex change and the test rows.
F2 (`haltnote`): two comments describe the routed field as Stuck-only although reroute rows carry it too; the fix is a rewording.
