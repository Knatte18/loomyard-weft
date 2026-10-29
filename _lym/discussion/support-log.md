# Support log — ide-seed-workspace-and-keys

## Interview

A darn run holds no interview: the decisions come from the board entry and a reading of `internal/ideengine`, `internal/vscode` and `fabricengine`'s hub workspace regeneration.

Findings that set the scope:

- The live hub workspace file splices the prime's `settings.json` verbatim, so once regenerated after the prime's seed it carries the prime's title and green, identical to the prime window.
  The prime's settings were written after the operator's report and the workspace file was regenerated hours later by a fabric verb; `lym ide seed` itself never regenerates it.
- The operator's user `keybindings.json` holds no `lym:begin` block.
  `lym ide spawn` seeds the block; `lym ide seed` explicitly leaves the file alone, and a test pins that.

## Rejected alternatives

- `terminal.integrated.sendKeybindingsToShell: true` in the workspace or worktree settings: it sends nearly every key binding to the shell, taking away VS Code's own terminal shortcuts, and is unproved here, where the key binding block is lym's existing mechanism.
- Removing VS Code's pane-focus commands through `terminal.integrated.commandsToSkipShell` entries: the keys would then reach the shell as raw Alt-arrows, but the setting's scope across windows is unclear and it duplicates what the user-level block already does.
- A workspace colour on both bars: it hides which project the workspace belongs to, against the package's title-bar-is-project scheme.
- A workspace colour from the palette: a task could pick it, so a task window could look like the workspace.
- Applying the look only in `ideengine` after regeneration: a later fabric-verb regeneration would strip it.
- Leaving beam workspace files as they are: a beam's workspace window would stay identical to the beam's own window.
- Regenerating the hub workspace on every seed, a task's included: a pinned contract has a task spawn leave the file's bytes alone, and a task seed has no reason to touch the hub file.

## Review rounds

_No rounds yet._

## Question ledger

- Which setting passes Alt+arrow through? Self-pick: the existing user keybindings block (D5); the failure was that seed never wrote it.
- Which colour does the workspace take, and where? Self-pick: a reserved non-palette colour on the activity bar, the project's colour kept on the title bar (D1).
- Do beam workspace files change too? Self-pick: yes, the same look with the beam's slug in the title (D2, D3).
- Does a task seed regenerate the hub workspace? Self-pick: no (D4).
