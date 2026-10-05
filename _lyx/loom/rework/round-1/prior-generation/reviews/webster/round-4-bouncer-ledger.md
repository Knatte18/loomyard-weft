---
round: 4
ledger:
  - key: hard-wrapped-go-comments
    rounds: [1, 2, 4]
    status: open
  - key: stale-config-menu-entry-in-overview
    rounds: [1]
    status: resolved
  - key: stringnode-missing-doc-comment
    rounds: [1]
    status: resolved
  - key: independent-clauses-on-one-line
    rounds: [2, 3, 4]
    status: open
---

Round 1 raised three `consistency` findings; two were resolved in round 2.

`hard-wrapped-go-comments` was resolved after round 3, and round 4 reopens it.
F1 to F5 are mid-clause breaks in boardcli, configcli, configsync and selfreportcli comments.

`independent-clauses-on-one-line` was new in round 2 and recurs in rounds 3 and 4.
Round 4's F1 and F6 are two clauses on one line, in boardcli and yamlengine.
Each round raises six BLOCKING findings at new sites, so the fixes land on cited lines while the same fault remains elsewhere in the diff.
