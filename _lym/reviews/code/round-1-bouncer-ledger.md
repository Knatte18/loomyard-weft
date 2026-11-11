---
round: 1
ledger:
  - key: overview-machinename-entry-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: filepath-unset-and-relative-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: userconfigdir-relative-xdg-empty-home-untested
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 raised three non-gating findings.
F1 is the stale `internal/machinename` entry in `docs/overview.md`.
F2 is the missing `FilePath` rows for the unset and relative-variable cases.
F3 is the missing `TestUserConfigDir` row for a relative `XDG_CONFIG_HOME` with an empty `HOME`.
All three are local corrections whose text the review already fixes.
