# Decision record — ide-seed-workspace-and-keys

## Goal

`lym ide seed` seeds the hub's VS Code workspace with its own window title and colour, so a workspace window is told apart from its worktrees at a glance, and seeds what the VS Code integrated terminal needs to pass Alt+arrow through to Reed's tmux bindings.

## Scope

- Every regeneration of a generated workspace file, the hub's `<hub>/<RepoName>.code-workspace` and each beam's, sets lym's window keys in the spliced settings to a workspace look: its own title and its own activity-bar colour.
- `lym ide seed` of the prime, with the prime's slug or with no slug run in the prime, regenerates the hub workspace file after writing the prime's config, as `lym ide spawn` of the prime does.
- `lym ide seed` seeds the lym block of key bindings into the user's VS Code `keybindings.json`, as `lym ide spawn` does, and reports the outcome.
- Out of scope: any change to the keybindings block's entries, to reed's tmux bindings, and to the look of a worktree window.

## Decisions

### D1 — The workspace look: project colour on the title bar, a reserved workspace colour on the activity bar

A workspace window keeps the project's colour on its title bar, the colour the spliced settings' title bar holds, the prime's green when they hold none, and takes a new reserved colour, `vscode.WorkspaceColor` (new), on its activity bar.
The reserved colour is outside the palette, so no worktree ever picks it.
Rationale: the package's colour scheme already puts the project's colour on the title bar and the window's own colour on the activity bar; a workspace following it reads as the prime's or the beam's project while no worktree window looks the same.

### D2 — The workspace title names the workspace

A workspace file's `window.title` is `<SHORTNAME>: <name> workspace`, `<name>` being the repository name for the hub file and the beam's slug for a beam file, and `<name> workspace` when the board has no shortname.
Rationale: the prime's title is `<SHORTNAME>: <prime slug>`, which for a hub whose prime is named after the repository is the same text; the suffix tells the two apart in the window switcher.

### D3 — The look is applied where the settings are spliced, in every regeneration

`fabricengine`'s regeneration applies the workspace look to the settings it splices, through a new `vscode` function that decodes the JSONC settings, sets lym's window keys and re-encodes them.
Settings that do not decode as a JSONC object are spliced verbatim with a warning, as a worktree's undecodable settings are left alone, and missing settings yield the look's keys alone.
Rationale: every path that writes the file, a fabric verb, `lym fabric reconcile`, a prime spawn or seed, then writes the same look, so no later regeneration strips it.
Bound: the spliced settings lose their comments, which a seeded worktree's `settings.json` has already lost at its own seed.

### D4 — Seeding the prime regenerates the hub workspace file

`ideengine.Seed` regenerates the hub workspace file when its slug is the prime, and returns a regeneration error as `ideengine.Spawn` does for the prime.
Seeding a task pair leaves the workspace file as it is.
Rationale: the workspace carried no colour because the prime was seeded after the file was last generated, and seed never wrote it.

### D5 — The Alt+arrow pass-through is the existing user keybindings block, now seeded by seed too

`ideengine.Seed` seeds the lym block of key bindings into the user's `keybindings.json` through the same seam `ideengine.Spawn` uses, and `lym ide seed` prints the outcome under `keybindings`, as `lym ide spawn` does.
The block binds Alt+Up, Alt+Down, Alt+Left and Alt+Right under `terminalFocus` to `workbench.action.terminal.sendSequence` with the xterm Alt-arrow sequences reed's tmux `M-Up`, `M-Down`, `M-Left` and `M-Right` bindings read, so the keys reach tmux instead of VS Code's own terminal pane commands.
A user-level key binding applies in every window, so the keys work in any hub opened in VS Code.
Rationale: the operator's `keybindings.json` holds no lym block, because only `lym ide spawn` seeds it and the operator seeds and opens windows without spawning.
Bound: seed now writes a user-level file outside the hub, as spawn already does; a file it cannot merge is left untouched and reported as skipped, never failing the seed.

## Constraints

- `PATTERN-fabric-git` and `PATTERN-cwd-resolution`: the hub workspace file is written only by `fabricengine`, through its existing regeneration.
- The workspace-look keys are the keys `vscode.WriteConfig` owns in a worktree's `settings.json`; one function sets them for both.
- Tests that call `ideengine.Seed` already point `ideengine.KeybindingsPath` at a temporary file, so no test writes the operator's `keybindings.json`.
- Docs in the same commit: the `vscode` package doc, `idecli`'s file header and `fabricengine`'s `doc.go` paragraph on `RegenerateHubWorkspace`.

## Auto-mode assumptions

- The board entry's "its own colour" means a colour no worktree window takes; the workspace keeps the project colour on its title bar so it still reads as the prime's or the beam's project.
- Beam workspace files get the same look as the hub file, since a beam's workspace window is otherwise identical to the beam's own window, the same complaint the entry makes about the hub.
- The VS Code setting the entry asks to find is the existing user-level key binding block, not a `terminal.integrated.*` setting.

## Open risks

- The key pass-through cannot be driven in VS Code from a test; that the keys switch Reed windows and sessions in the VS Code terminal is checked by the operator after `lym ide seed`.
- `terminal.integrated.sendKeybindingsToShell` set by the operator would not stop the block, but a user binding of the same keys placed after the block in `keybindings.json` overrides it.

## Acceptance criteria

- D1, D2, D3: a regenerated hub workspace file's settings carry the prime's keys, the title `<SHORTNAME>: <RepoName> workspace`, the prime's green on the title bar and `vscode.WorkspaceColor` on the activity bar; a beam file carries the beam's title-bar colour and `<SHORTNAME>: <beam> workspace`.
- D3: missing prime settings yield a workspace file whose settings hold the look's keys alone; undecodable settings are spliced verbatim; an unchanged regeneration rewrites no bytes.
- D1: `vscode.WorkspaceColor` is no palette colour, so no picker returns it.
- D4: `lym ide seed` of the prime writes the hub workspace file; seeding a task pair leaves its bytes as they were.
- D5: `ideengine.Seed` creates the keybindings block in a missing `keybindings.json` and returns the outcome, and `lym ide seed` prints it under `keybindings`.
