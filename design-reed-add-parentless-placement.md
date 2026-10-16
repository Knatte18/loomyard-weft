# reed add defaults to pane-era placement and splits the selvage shell window

`reed-add-parentless-placement` · Task · bug, reed · running · Plan-Write

Imported from [issue #673](https://github.com/Knatte18/loomyard/issues/673).

## Seen

In the hub prime, the orch ran `lyx reed add --name stencil-audit --cmd "claude --resume <id> ..."` with no `--anchor` and no `--parent`.
The default anchor is `below-parent`, and with no parent the new strand `ly:stencil-audit` was split into tmux window `shell` (13.0), beside the selvage bash pane (13.1).
The `shell` window must hold only the selvage terminal.

Workaround: pass `--anchor own-window` to every parentless `lyx reed add`; recorded in CLAUDE.md under Workarounds with this issue number, and removed with the fix.

## Operator decision (2026-10-16)

The `below-parent` default is left over from before reed used tmux windows, when strands were panes in one window.
It is obsolete and must be fixed.
The task also searches the code, stencils and docs for other leftovers of that pane-era behaviour and fixes them.

## Scope

- `lyx reed add` no longer defaults to splitting a pane into an existing window: a new strand gets the placement that matches today's window-per-strand layout, and it never lands in the selvage `shell` window.
- A sweep for other pane-era assumptions (placement defaults, layout code, help texts, stencils, docs and tests that still describe strands as panes split under a parent) and a fix for each one found.
- The CLAUDE.md workaround for #673 is removed in the same change.
