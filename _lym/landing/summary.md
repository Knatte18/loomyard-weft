# lym ide seed colours the VS Code workspace and seeds the Alt-arrow keys

## What changes

- **Workspace look.**
  Every generated workspace file, the hub's `<hub>/<RepoName>.code-workspace` and each beam's, now splices its settings through `vscode.WorkspaceSettings`.
  It sets `window.title` to `<SHORTNAME>: <name> workspace`, after the repository or the beam, and puts the new reserved colour `vscode.WorkspaceColor` (`#455a64`, blue-grey, outside the palette) on the activity bar.
  The title bar keeps the project colour the spliced settings hold, the prime's green when they hold none.
  A workspace window therefore still reads as its project but no longer looks the same as the prime or the beam window whose settings it carries.
  The look is applied inside `fabricengine`'s regeneration, so every path that rewrites the file, a fabric verb, reconcile, a prime spawn or seed, writes the same look.
- **Seed writes the workspace.**
  `ideengine.Seed` of the prime now regenerates the hub workspace file and returns any error from it, as `Spawn` of the prime already did.
  A task seed leaves the file alone.
  `Spawn` and `Seed` now share one `seed` path; `Spawn` only adds the launch.
- **Alt+arrow pass-through.**
  The mechanism already existed: the `// lym:begin` block in the user's `keybindings.json` binds Alt+Up/Down/Left/Right under `terminalFocus` to `workbench.action.terminal.sendSequence` with the xterm Alt-arrow sequences reed's tmux `M-*` bindings read.
  Only `lym ide spawn` wrote it, and the operator's `keybindings.json` held no block, so VS Code's own terminal pane commands took the keys.
  `lym ide seed` now seeds the block too and prints the outcome under `keybindings`, like `lym ide spawn`.
  A user-level binding applies in every window, so the keys reach reed in any hub.

## Design decisions

- `slugVerbCommand` in `internal/idecli/cli.go` now takes an action that returns extra output fields, merged beside `slug`, so `seed` can report its keybindings outcome; `open` returns none.
- `vscode.WorkspaceSettings` decodes the JSONC settings and re-encodes them, so comments in the spliced settings are dropped.
  A seeded worktree's `settings.json` has already lost its comments at its own seed.
  Settings that do not decode as a JSON object are spliced verbatim with a warning, matching how `WriteConfig` leaves an undecodable `settings.json` alone.
- The look-setting code that `WriteConfig` used inline is now `applyLook`, shared by the worktree and the workspace paths.
- batten's `Seed-IDE` row runs `lym ide seed {slug}`, so every task run now also keeps the user's keybindings block in place.
  The write is idempotent, an unchanged file is not rewritten, and a file it cannot merge is skipped without failing the row.
  The `seed_ide_command` comment in `internal/battenshed/template.yaml` and `docs/overview.md` now say so in place of "without the user's key bindings".
  No test runs a real `lym ide seed` through batten; its lifecycle tests use marker commands.

## Tests

- `TestWorkspaceSettings` (tier0) covers prime and beam settings, JSONC with a BOM, missing and comment-only settings, and undecodable settings returned unchanged; `TestWorkspaceColorIsNoPaletteColor` pins the reserved colour outside the palette.
- `TestRegenerateHubWorkspace/SplicesTheBeamsOwnSettingsIntoItsFileAndThePrimesWhenItHasNone` now tells the spliced source apart by a key lym does not own, and asserts each file's workspace title, title-bar colour and activity-bar colour.
- `TestSpawnScenario` gains a step where a prime seed writes the hub workspace file with its workspace title and launches nothing.
  The task step now expects `Seed` to create the keybindings block.
  The splice step asserts the workspace look in place of a verbatim splice.
- `TestRunCLI_SpawnScenario`'s prime seed step asserts the `keybindings` outcome in the JSON result.

## Not verified here

The key pass-through cannot be driven in VS Code from a test.
To check it, run `lym ide seed` once, then press Alt+arrow in a VS Code terminal attached to reed and confirm it switches windows and sessions.
