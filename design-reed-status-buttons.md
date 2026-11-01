# Configurable action buttons on the reed status bar

`reed-status-buttons` · Task · feature, reed, config, batten · draft

Depends on: [`kde-window-placement`](design-kde-window-placement.md)

## Seen

tmux status lines support clickable ranges (`#[range=user|<name>]` with a `MouseDown1Status` binding), and reed already uses them for its window buttons and the `follow` button, each running a command through `run-shell`.
Those buttons are fixed in code; the operator has no quick way to run an action on a run from where they watch it.

## Design (operator, 2026-11-01)

The action buttons are config, not code: adding a button is one entry in reed's config, with no build.

```yaml
buttons:
  - label: HALT
    run: lym tender pause {slug}
    show: run
  - label: CANCEL
    run: lym batten cancel {slug}
    show: run
    confirm: "Cancel {slug}?"
  - label: code
    run: lym ide spawn {slug}
    show: run
  - label: term
    run: lym reed window {worktree}
  - label: distill
    run: lym orch distill
    show: orch
    confirm: "Distill the orch?"
```

- `run` is a shell command, run in the session's worktree; lym fills the placeholders `{slug}`, `{worktree}`, `{beam}` and `{session}` at click time.
- `show` names where the button appears: `run` (a task run's session), `orch`, `beam`, or `all`, the default.
- `confirm` puts tmux's plain y/n `confirm-before` prompt, with the given text, in front of the command; buttons that cannot be undone carry it.
- A failing command shows a short tmux message on the status line; its output goes to a log.
- Layering: the hub's config holds the default set, and a machine-local override may add buttons of its own.
- The hub's default set holds the buttons the operator named: HALT (`lym tender pause <slug>`, which holds agent and network work and interrupts the running turns at once; `lym batten pause` waits for a producer boundary, so it is no halt), CANCEL (kept apart from HALT, behind `confirm`), open the pair in VS Code, one more terminal on the same worktree (`lym reed window <worktree-dir>`), a new empty agent (`lym shuttle open --role <role> --model opus[high]`), and `lym orch distill` and `lym orch refresh` on the orch session, both behind `confirm`.
- Later buttons are entries too, such as the board agent of note `board-agent-button`.

## Placement (operator, 2026-10-30)

The action buttons go on a third status line of their own, never mixed with the window buttons (line 1) or the session names (line 2).
The `follow` button leaves line 2 and joins the action row as a built-in entry.
Line 2 also renders the prime's adopt-check mark right-aligned before the clock (`Engine.SetCheckMark`); the action row leaves it there.

## Open

- Which role the new-agent button opens, and whether the role name is asked for or fixed.
