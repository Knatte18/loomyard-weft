---
round: 2
ledger:
  - key: log-level-env-reset-by-root-hook
    rounds: [1]
    status: resolved
  - key: boot-refusals-unreached-at-planned-site
    rounds: [1]
    status: resolved
---

Round 1 raised two findings: the root hook reset the level `LYX_LOG_LEVEL` set, and the boot refusals were shadowed and untested at their planned site.
Round 2 found both fixed and raised nothing new.
