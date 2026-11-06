---
round: 2
ledger:
  - key: driftbaseline-engine-name-bare-literal
    rounds: [1]
    status: resolved
  - key: skipfileargument-eats-enclosing-quote
    rounds: [1]
    status: resolved
  - key: skipfileargument-breaks-backslash-continuation
    rounds: [1]
    status: resolved
  - key: judge-session-mark-order-unpinned
    rounds: [1]
    status: resolved
  - key: volatile-keys-hub-only-refusal-untested
    rounds: [1]
    status: resolved
  - key: expect-pass-volatile-recording-untested
    rounds: [1]
    status: resolved
  - key: quoted-shell-string-wording-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's six findings are resolved: the review's focus verdicts show each fixed with a pinning test where one was asked for.
Round 2 adds F1 (LOW): the MigrateConfig docs say any argument inside a quoted shell string stays, but only one that ends the string does, and no row pins the removal of a non-final one.
