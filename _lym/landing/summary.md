# lym ide seed: colour the workspace, seed the Alt+arrow keys

A hub opened as a VS Code workspace carried no colour, and Alt+arrow did not switch Reed windows and sessions in its integrated terminal.
`lym ide seed` wrote a worktree's config but never the workspace file, and never the user key bindings that forward the keys to tmux.

## What changes

- Every regeneration of a generated workspace file, the hub's `<RepoName>.code-workspace` and each beam's, now sets a workspace look in the settings it splices.
  `window.title` becomes `<SHORTNAME>: <name> workspace`, with `<name>` the repository name or the beam's slug, and `<name> workspace` when the board has no shortname.
  The title bar keeps the project colour, the prime's green when the settings hold none, and the activity bar takes the new `vscode.WorkspaceColor`, a blue-grey outside the palette so no worktree window looks the same.
- `lym ide seed` of the prime, by slug or with no slug run in the prime, regenerates the hub workspace file after writing the prime's config, as `lym ide spawn` does.
  Seeding a task pair leaves the workspace file alone.
- `lym ide seed` seeds the lym block into the user's VS Code `keybindings.json`, as spawn does, and prints the outcome under `keybindings`.
  The block binds the four Alt-arrow keys under `terminalFocus` to `workbench.action.terminal.sendSequence` with the sequences Reed's tmux `M-Up`, `M-Down`, `M-Left` and `M-Right` bindings read.
  It is user-level, so the keys work in any hub opened in VS Code.
  The block itself is unchanged; no `terminal.integrated.*` setting is involved.

## Design decisions

- The look is applied in `fabricengine`'s regeneration through the new `vscode.WorkspaceSettings`, so a fabric verb, `lym fabric reconcile` and a prime spawn or seed all write the same look and no later regeneration strips it.
  Settings that do not decode as a JSONC object are spliced verbatim with a warning; missing settings yield the look's keys alone.
  Spliced settings lose their comments, as a seeded worktree's `settings.json` already does.
- `WorkspaceSettings` and a worktree's `settings.json` share one `applyLook`, so both set the same keys.
- `ideengine.Spawn` and `ideengine.Seed` share one internal `seed`; `Spawn` adds the launch.
  `Seed` now returns the `KeybindingsResult` beside the error, and `idecli`'s `slugVerbCommand` prints the fields its action returns, which is why `open` wraps `OpenTask`.
- Seed now writes a user-level file outside the hub, as spawn already does.
  A file it cannot merge is left untouched and reported as skipped, never failing the seed.
- The batten `Seed-IDE` row runs `lym ide seed`, so a batten run now seeds the key bindings too; `battenshed/template.yaml` and `docs/overview.md` say so.

## Check by hand

Not testable from Go, and not run:

- Run `lym ide seed` in the prime, open the hub workspace in VS Code, and confirm the workspace title and blue-grey activity bar.
- In that VS Code terminal, confirm Alt+arrow switches Reed windows and sessions.
  If the operator has a user binding of the same keys after the lym block in `keybindings.json`, it overrides the block.
