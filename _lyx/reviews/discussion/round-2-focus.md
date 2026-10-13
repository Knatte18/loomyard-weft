---
round: 2
focus:
  - "Check whether the decision record's Constraints place `internal/selfexec`'s closed import set under `PATTERN-leaf-packages` (background entry plus a `leaf_enforcement_test.go` using `scankit.AssertImportAllowlist`), and whether `internal/statuspane`'s import list is stated consistently with D8."
  - "Check whether D2 gives every `new-window` insertion, hidden and no-shell fallback alike, an explicit target window, and whether D5's newest-is-highest-index rule and D6's single conditional step still hold under that order, including on psmux."
  - "Check whether D10's rename and `move-window` migration step states what happens when either fails or races a concurrent open or attach, and what bounds it."
  - "Check whether D1's replace-selection rule and D5's bound that an automatic switch never moves a viewer off the shell agree for a replace of a top-anchored status strand whose window is the shell."
---

Focus entries name where to look; they do not cap severity.
