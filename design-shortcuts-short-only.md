# Shortcuts are short forms only

`shortcuts-short-only` · Task · enhancement, shortcut, loom, reed

## Seen (operator, 2026-10-25)

The command surface carries long-form shortcuts beside the short ones: `lyx attach` beside `lyx a`, `lyx pause` beside `lyx p`, `lyx resume` beside `lyx r`.
A top-level `lyx start` runs `lyx loom start`, which nobody runs since batten starts every loom run.

## Design (operator, 2026-10-25)

Shortcuts are short forms only:

| Shortcut | Runs |
|---|---|
| `lyx a` | `lyx reed attach` |
| `lyx p` | `lyx tender pause` |
| `lyx r` | `lyx tender resume` |

- The long forms `lyx attach`, `lyx pause` and `lyx resume` are removed.
- The top-level `lyx start` is removed; `lyx loom start` stays.
- Docs, stencils and the command index stop naming the removed forms.
