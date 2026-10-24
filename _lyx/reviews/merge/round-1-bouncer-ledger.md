---
round: 1
ledger:
  - key: verified-tree-round-gate-sites
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding: the `PATTERN-verified-tree` background file names only the fixer fork as the `round` gate's site, while the parent's cold fixer is a second site.
The fix is a local site-list correction.
