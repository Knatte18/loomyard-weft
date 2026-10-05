---
round: 3
ledger:
  - key: hard-wrapped-go-comments
    rounds: [1, 2]
    status: resolved
  - key: stale-config-menu-entry-in-overview
    rounds: [1]
    status: resolved
  - key: stringnode-missing-doc-comment
    rounds: [1]
    status: resolved
  - key: independent-clauses-on-one-line
    rounds: [2, 3]
    status: open
---

Round 1 raised three `consistency` findings; two were resolved in round 2.
`hard-wrapped-go-comments` is resolved: round 3 raised no fixed-column wrap in changed lines.

`independent-clauses-on-one-line` was new in round 2 and recurs in round 3 as F1-F6.
Each is BLOCKING and cites `PATTERN-comment-line-breaks`, across yamlengine, configengine, configcli, boardengine, boardcli and two test files.
The sites differ from round 2's, which shows the fixes landed on the cited lines but the pattern was not swept across the diff.
