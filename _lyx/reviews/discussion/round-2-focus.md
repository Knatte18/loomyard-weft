---
round: 2
focus:
  - "In decision-record.md, D5 skips every directory below the worktree root that holds its own go.mod, and D2 says a package-level caller never weakens because D3 resolves it exactly. Settle whether any caller lost to D5 (a nested module importing a root package, or any caller of a subject declared in a nested module) is a package-level caller the current scan reports as blocking, and whether D5 or D2 states what bounds that loss."
  - "In decision-record.md, D9 says a finding whose fix changes targets or a verify command must name a fix every validation check admits, and a fix such a check blocks is not a legitimate finding. Settle whether a reviewer who sees a real defect but only a check-blocked fix is told to drop the defect, and whether D9 states what that lets through and what bounds it."
  - "In decision-record.md, D1 stops reporting calls through an interface for a re-signed method, which today surface as informational caller-uncovered entries. Settle whether D1 states what still catches the resulting break."
  - "In decision-record.md, D6 builds the nested-module set from go.mod files on disk plus Create targets minus Delete targets through the card, while a card's Rename group can move a go.mod. Settle whether D7's card gate and D8's verdict give the same result before and after a renaming card."
---

The two BLOCKING design questions come first; the other two are lower-stakes checks of the same attack-surface rule.
Both Open-risks requirements and D8's `go -C` base are already named in the findings and are not re-promoted here.
