---
round: 1
focus:
  - "internal/desktophook/window.go: does the command run only once the new window is mapped, or after a short wait, as the Goal requires? Settle how a failing, hanging or missing command is handled so it never breaks or blocks the window open."
  - "internal/desktophook/config.go and template.yaml: is the machine-local key unset by default, and does an unset key run nothing? Settle that the key is machine-scoped and that the command string is split and executed safely, with no shell interpretation surprises."
  - "internal/ideengine/spawn.go and internal/reedcli/window.go: is the hook called after the terminal window (`lym reed window`) and the VS Code window (`lym ide`) in every open path, and in no other path? Settle whether any opener the Goal names is missed."
  - "Goal says lym knows nothing about KDE: settle that no KDE, KWin or Plasma specific code or naming entered the diff outside docs and comments."
  - "internal/configreg, configcli, configengine edits and pattern/PATTERN-config-layers.md: do they register the new key consistently with the repository's config-layer rules, and do docs/overview.md and PATTERN land in the same diff as the rules require?"
  - "Test economy: do the new desktophook, reedcli and ideengine tests duplicate each other's coverage, and does any added behavior lack a test?"
  - "Commit messages and any Pattern-Departure trailers: settle whether each trailer names a real entry that forbids the answer key's way."
---

Seed focus for round 1, drawn from the decision record's Goal and the diff stat of the two commits in the review range.
