---
round: 1
focus:
  - "internal/vscode/workspace.go and internal/fabricengine/hubworkspace.go: does the workspace look set the title as `<SHORTNAME>: <name> workspace` (hub: repository name, beam: slug, no-shortname fallback), keep the project colour on the title bar with the prime green default, and put `WorkspaceColor` on the activity bar, for both hub and beam files?"
  - "Settings splicing in fabricengine: are missing settings, undecodable JSONC (verbatim splice with a warning) and an unchanged regeneration (no bytes rewritten) each handled, and is the key set shared with `vscode.WriteConfig` through one function rather than duplicated?"
  - "internal/ideengine/spawn.go: does Seed regenerate the hub workspace file only for the prime, return a regeneration error as Spawn does, and leave a task pair's workspace file untouched? Is the prime detected for both an explicit slug and no slug?"
  - "internal/ideengine/spawn.go and internal/idecli/cli.go: does Seed seed the user keybindings block through the same seam as Spawn, never fail the seed on an unmergeable file (reported as skipped), and does `lym ide seed` print the outcome under `keybindings`? Do tests keep `KeybindingsPath` on a temporary file so none writes the real one?"
  - "internal/vscode/color.go: is `WorkspaceColor` outside the palette so no picker can return it, and is that covered by a test?"
  - "Scope: the diff touches docs/overview.md and internal/battenshed/template.yaml; does the decision record license each change? Do docs land in the same diff for the vscode package doc, idecli file header and fabricengine doc.go, and is the keybindings block, reed's tmux bindings and the worktree window look left unchanged?"
  - "Test economy: do the new tests in workspace_test.go, hubworkspace_integration_test.go and spawn_hub_integration_test.go duplicate each other's coverage, and does every added behavior have a test?"
---

Seed read from the board entry and decision record; no prior round exists.
