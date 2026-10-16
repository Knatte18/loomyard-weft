---
round: 2
focus:
  - "Compile order across cards 2 and 5: for every test member that references `render.AnchorTop` or `render.AnchorBelowParent` (the render types test, the reedengine attach tests and the attach-geometry integration fixture), settle whether some card at or before card 5 lists it, so the render and reedengine test binaries still build at card 5's commit."
  - "Stale stack wording: settle whether every non-exempt file the plan's third `## verify:` grep scans, and every file header naming the below-parent or top-anchored stack (render policy file header, reedcli smoke test headers), is rewritten by some card."
  - "Emptied files: settle whether each test file left with no remaining test function after a card's deletions is itself deleted whole, rather than left with unused imports."
  - "Edit-target honesty: settle whether each test listed under an Edit target on card 5 is actually changed by that card's Intent."
---

Each entry restates the site it depends on and cites no earlier round.
