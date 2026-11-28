---
round: 1
focus:
  - "internal/shedcli/table_test.go, TestRecipes_KeySet: does the rewritten comparison still check, for every table case, what the test's name and doc comment state, with no limit looser than at the review base?"
  - "internal/shedcli and the repository documents: does any comment, doc or spec still describe the removed `names` accessor, or two accessors on the recipes table, and do the changed file headers match the code as the diff leaves it?"
  - "_lym/landing/summary.md: does the diff and the current code bear out each claim, including the sole-caller claim, the reason given for deleting rather than wiring a production caller, and the `## Merge danger` door and blast radius?"
---

The range is one commit that deletes an unexported function and rewrites its one test.
The entries point at where that kind of change can leave something behind.
