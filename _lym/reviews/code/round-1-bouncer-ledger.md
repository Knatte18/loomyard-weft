---
round: 1
ledger:
  - key: redirect-amp-skip-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: quoted-digit-operand-dropped-as-fd
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: redirect-behaviours-without-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all in the redirect handling of the Burler fork git guard.
F1 (`redirect-amp-skip-unpinned`) asks for a test row pinning the `&` skip after a redirect operator.
F2 (`quoted-digit-operand-dropped-as-fd`) is a quoted or escaped digit operand wrongly dropped as a file descriptor.
F3 (`redirect-behaviours-without-rows`) is three redirect behaviours with no pinning rows.
