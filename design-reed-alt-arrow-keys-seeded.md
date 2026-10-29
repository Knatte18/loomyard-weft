# Reed navigation keys, Alt+arrows, are seeded by lym

`reed-alt-arrow-keys-seeded` · Note · enhancement, reed, config

## Seen

In the loomyard hub opened as a VS Code workspace, Alt+arrow keys do not switch windows and sessions in Reed, which they should.
In a Konsole window beside it they do (operator, 2026-10-29): the same session worked in no Konsole and failed in the bash terminal inside VS Code, so the layer that swallows the keys is VS Code's integrated terminal, not tmux or the desktop.

## Idea

Lym seeds what the VS Code terminal needs to pass Alt+arrow through to tmux, in the settings `lym ide seed` already writes for a worktree, so the keys work in any hub opened in VS Code without manual setup (see `workspace-ide-seed`).
Which VS Code setting does it, such as `terminal.integrated.sendKeybindingsToShell` or a `commandsToSkipShell` entry for the keys, is not yet known and is the first thing to find.
