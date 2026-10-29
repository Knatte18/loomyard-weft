# lym ide seed colours the VS Code workspace and passes Alt+arrow through to Reed

`ide-seed-workspace-and-keys` · Task · enhancement, ide, workspaces, reed, config · draft

## Goal

`lym ide seed` seeds a VS Code workspace like a worktree, and seeds what the VS Code terminal needs to pass Alt+arrow through to Reed's tmux.

## Seen

- The operator opened the loomyard hub as a VS Code workspace and it carries no colour.
  `lym ide seed` writes the VS Code config, window title and colours of the prime or a task worktree, but nothing for the workspace itself.
- In that workspace, Alt+arrow keys do not switch windows and sessions in Reed.
  In a Konsole window they do (operator, 2026-10-29), so the layer that swallows the keys is VS Code's integrated terminal, not tmux or the desktop.

## Scope

- Seed the workspace like a worktree: its own colour, window title and the rest of the config `lym ide seed` writes for a worktree, so a workspace window is told apart from its worktrees at a glance.
- Seed the VS Code setting that passes Alt+arrow through to the shell, so the Reed keys work in any hub opened in VS Code.
  Which setting does it, such as `terminal.integrated.sendKeybindingsToShell` or a `commandsToSkipShell` entry, is the first thing to find, and is proved by the keys working in the VS Code terminal.
