# PR-Gate review: board-labels-and-skills (PR #374)

Everything in the board goal is covered; one change is rejected.

## Finding: the interactive `lyx config` menu must stay, as `lyx config menu`

D9 deleted the interactive module picker (`internal/configcli/menu.go`) on auto-assumption A1 ("nobody depends on it"), which no operator decided.
The operator wants both: bare `lyx config` keeps the new cobra listing of its subcommands, so the CLI tree invariant holds, and the picker survives as a subcommand.

- Restore the picker as `lyx config menu`: the numbered menu of known modules that opens the chosen module's editor, as bare `lyx config` did before.
- Bare `lyx config` lists `menu` among its subcommands, with a `Short` that says it picks a module interactively.
- `lyx config menu` is covered by the CLI tree walk like every other verb, and `internal/configcli`'s `doc.go` and the help say where the picker went.

## Finding: the editor fallback is vi, which the operator cannot use

With neither `$VISUAL` nor `$EDITOR` set, `lyx config <module>` (and the menu) opens vi; the operator tried it on 2026-10-05 and could not use it.
Change the fallback order after `$VISUAL` and `$EDITOR` to: `code --wait` when `code` is on PATH, then `nano`, and `vi` only when neither exists (notepad stays the Windows fallback).
State the order in the command's help and in `internal/configcli`'s `doc.go`.

Nothing else in the PR changes.
