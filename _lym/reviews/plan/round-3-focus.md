---
round: 3
focus:
  - "Card 2 (`_lym/plan/02-buttons-config-keys.md`) and the reed config template: the reed templates ship the `buttons.task` and `buttons.beam` groups as lists of mappings, and `configengine.Load` runs `yamlengine.MissingKeys` over the template's list elements unless the paths are declared open maps. Settle whether the plan declares `buttons.task`, `buttons.beam` and `buttons.prime` as open maps in both the reed Spec and the `internal/configreg` `Modules` entry (which `TestModules_SeedOnlyAndHubWideFlags` compares with the Spec), and whether every card 2 test row about a hub-base or worktree override (a mapping group, a shorter list, `buttons: null`, the local-layer refusal naming `buttons.task`) is reachable under that declaration."
  - "Cards 2 and 9: `Extends` is for adding to a ref without changing anything already there. Settle whether any `Extends` target in these cards (the `docs/overview.md` tree line for `internal/desktophook/`, the tmux-features sentence of `internal/toolfloor/doc.go`) has existing content rewritten, and whether the corrected typing passes `redundant-file-target` and `redundant-package-target` in the spec's validation-checks section."
  - "Card 5 and `internal/reedengine/bar_integration_test.go`: card 5 makes the status bar three lines. Settle whether any comment or doc sentence in a file the card touches still says two-line and no card names it."
---

Focus covers the round 2 defect set and the edits that answer it.
