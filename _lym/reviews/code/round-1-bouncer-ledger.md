---
round: 1
ledger:
  - key: writer-of-stray-git-not-found-or-fixed
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: blame-without-evidence-of-writing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: classify-processes-test-lacks-keep-directive
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: finding-text-claims-common-binary-is-writer
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: stop-drops-queued-create-event
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: later-sighting-overwrites-earlier
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-behaviors-without-pinning-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: quoteargv-cuts-by-bytes
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, F1 to F8, mapped one to one onto the keys above.
Three are BLOCKING: the missing writer fix (F1), the blame rule that fails packages that did not write the entry (F2),
and the missing `//testtiming:keep` on `TestClassifyProcesses` (F3).
F4 is a MEDIUM wording fix; F5 to F8 are LOW or NIT.
No earlier ledger exists.
